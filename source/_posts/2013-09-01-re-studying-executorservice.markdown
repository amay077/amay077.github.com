---
layout: post
title: "ExecutorService の復習"
date: 2013-09-01 19:52
comments: true
categories: [Java,Android]
---
Android で非同期処理っていうと、真っ先に ``AsyncTask`` が出てくるんですが、なるべくなら Java 標準のマルチスレッドAPI である ExecutorService を使った方が良いと思ってます。
<!--more-->
* (2007年の記事だけど) [Java技術最前線 - 「Java SE 6完全攻略」第49回 Concurrency Utilitiesの変更点 その1：ITpro](http://itpro.nikkeibp.co.jp/article/COLUMN/20071001/283395/)

Android で初めて Java を書いたので細かい仕様がよく分からず、勉強がてら動きを確認してみました。

## 1. 1つのワーカスレッドに実行させる

``newSingleThreadExecutor`` でワーカスレッドを一つ持つ Executor を生成して、2つのタスク(=非同期で実行させる処理)を順に実行。

```java
public void singleThreadExecutorBasicTest() throws Exception {
	final ExecutorService executor = Executors.newSingleThreadExecutor();
	
	Log.d(TAG, "Primary ThreadID:" + Thread.currentThread().getId());
	executor.submit(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task A. ThreadId:" + Thread.currentThread().getId());
		}
	});

	executor.submit(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task B. ThreadId:" + Thread.currentThread().getId());
		}
	});
}
```

### 出力

>08-29 17:18:12.891: D/ExecutorTest(391): Primary ThreadID:4943<br/>
08-29 17:18:12.891: D/ExecutorTest(391): Run task A. ThreadId:4944<br/>
08-29 17:18:12.891: D/ExecutorTest(391): Run task B. ThreadId:4944<br/>

A→B の順で（=submit した順で）実行される。
Primary と task で ThreadID が異なる、2つの task は同じ ThreadID であることに注目。

## 2. 最初のタスクに時間がかかったら？

タスクA の実行に時間がかかる場合、タスクB はどうなる？

```java
public void singleThreadExecutorHeavyWorkTest() throws Exception {
	final ExecutorService executor = Executors.newSingleThreadExecutor();
	
	Log.d(TAG, "Primary ThreadID:" + Thread.currentThread().getId());

	executor.submit(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task A start. ThreadId:" + Thread.currentThread().getId());
			
			// Wait
			try { Thread.sleep(3000); } catch (InterruptedException e) { } 
			
			Log.d(TAG, "Run task A end.");
		}
	});

	executor.submit(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task B. ThreadId:" + Thread.currentThread().getId());
		}
	});
}
```

### 出力

>08-29 17:22:04.288: D/ExecutorTest(1511): Primary ThreadID:5063<br/>
08-29 17:22:04.288: D/ExecutorTest(1511): Run task A start. ThreadId:5064<br/>
08-29 17:22:07.291: D/ExecutorTest(1511): Run task A end.<br/>
08-29 17:22:07.291: D/ExecutorTest(1511): Run task B. ThreadId:5064<br/>

A→B の順で実行される。シングルスレッドなので、並列に処理されることはない。

## 3. ThreadPoolExecutor を使ったら？

``Executors.newSingleThreadExecutor()`` の代わりに ``Executors.newFixedThreadPool(2)`` としてみる。これによりワーカスレッドを2つ使う Execurot が生成される。

```java
public void threadPoolExecutorHeavyWorkTest() throws Exception {
	final ExecutorService executor = Executors.newFixedThreadPool(2);
	
	Log.d(TAG, "Primary ThreadID:" + Thread.currentThread().getId());

	executor.submit(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task A start. ThreadId:" + Thread.currentThread().getId());
			
			// Wait
			try { Thread.sleep(3000); } catch (InterruptedException e) { } 
			
			Log.d(TAG, "Run task A end.");
		}
	});

	executor.submit(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task B. ThreadId:" + Thread.currentThread().getId());
		}
	});
}
```

### 出力

>08-29 17:29:01.731: D/ExecutorTest(3017): Primary ThreadID:5255<br/>
08-29 17:29:01.731: D/ExecutorTest(3017): Run task A start. ThreadId:5256<br/>
08-29 17:29:01.731: D/ExecutorTest(3017): Run task B. ThreadId:5257<br/>
08-29 17:29:04.725: D/ExecutorTest(3017): Run task A end.<br/>

タスクA と タスクB で ThreadID が異なる事に注目。
スレッドが２つ使えるので、タスクA の終了を**待たず**にタスクB が実行される。というか、タスクA から始まる保証もない。

## 3. SingleThreadScheduledExecutor を使ってみる

話題を変えて、タスクの実行時間を制御できる Scheduled系 の Executor を使ってみる。

```java
public void singleThreadScheduledExecutorBasicTest() throws Exception {
	final ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();

	Log.d(TAG, "Primary ThreadID:" + Thread.currentThread().getId());
	executor.schedule(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task A. ThreadId:" + Thread.currentThread().getId());
		}
	}, 5, TimeUnit.SECONDS);

	executor.schedule(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task B. ThreadId:" + Thread.currentThread().getId());
		}
	}, 3, TimeUnit.SECONDS);
}
```

### 出力

>08-29 17:36:16.125: D/ExecutorTest(3547): Primary ThreadID:5285<br/>
08-29 17:36:19.128: D/ExecutorTest(3547): Run task B. ThreadId:5286<br/>
08-29 17:36:21.130: D/ExecutorTest(3547): Run task A. ThreadId:5286<br/>

開始から3秒後にタスクBが、開始から5秒後にタスクAが実行される。
submit した順は関係ないことに注意。

## 4. スケジュールされた時間に別のタスクが実行中だったら？

3秒後に実行されるタスクBの処理が終わらない時、5秒後に実行されるタスクAはどうなる？

```java
public void singleThreadScheduledExecutorHeavyWorkTest() throws Exception {
	final ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();
	Log.d(TAG, "Primary ThreadID:" + Thread.currentThread().getId());
	executor.schedule(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task A. ThreadId:" + Thread.currentThread().getId());
		}
	}, 5, TimeUnit.SECONDS);

	executor.schedule(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task B start. ThreadId:" + Thread.currentThread().getId());
			
			// Wait
			try { Thread.sleep(10000); } catch (InterruptedException e) { } 
			
			Log.d(TAG, "Run task B end.");
		}
	}, 3, TimeUnit.SECONDS);
}
```

### 出力

>08-29 17:44:18.139: D/ExecutorTest(4737): Primary ThreadID:5456<br/>
08-29 17:44:21.142: D/ExecutorTest(4737): Run task B start. ThreadId:5457<br/>
08-29 17:44:31.143: D/ExecutorTest(4737): Run task B end.<br/>
08-29 17:44:31.143: D/ExecutorTest(4737): Run task A. ThreadId:5457<br/>

5秒後とスケジュールされたタスクAだが、タスクB が終わるまで待たされる。シングルスレッドなので。

## 5. そこで ScheduledThreadPoolExecutor を使ってみる

``Executors.newSingleThreadScheduledExecutor()`` の代わりに ``Executors.newScheduledThreadPool(2)`` としてみる。

```java
public void threadPoolScheduledExecutorHeavyWorkTest() throws Exception {
	final ScheduledExecutorService executor = Executors.newScheduledThreadPool(2);
	Log.d(TAG, "Primary ThreadID:" + Thread.currentThread().getId());
	executor.schedule(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task A. ThreadId:" + Thread.currentThread().getId());
		}
	}, 5, TimeUnit.SECONDS);

	executor.schedule(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task B start. ThreadId:" + Thread.currentThread().getId());
			
			// Wait
			try { Thread.sleep(10000); } catch (InterruptedException e) { } 
			
			Log.d(TAG, "Run task B end.");
		}
	}, 3, TimeUnit.SECONDS);
}
```

>08-29 17:48:28.536: D/ExecutorTest(5439): Primary ThreadID:5558<br/>
08-29 17:48:31.550: D/ExecutorTest(5439): Run task B start. ThreadId:5559<br/>
08-29 17:48:**33**.542: D/ExecutorTest(5439): Run task A. ThreadId:5560<br/>
08-29 17:48:41.550: D/ExecutorTest(5439): Run task B end.<br/>

スレッドが２つ使えるので、タスクA は、スケジュール通り（タスクBの終了を待たずに submit してから5秒後に実行される。

## 6. Scheduled系Executor の Timer 的機能を使ってみる

``schedule()`` は、一発だけ(Javascript の ``setTimeout`` みたいな)、繰り返し処理するには、``scheduleAtFixedRate`` か ``scheduleWithFixedDelay`` を使う。
まずは ``scheduleAtFixedRate `` から。

```java
public void singleThreadScheduledExecutorTimerBasicTest() throws Exception {
	final ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();

	Log.d(TAG, "Primary ThreadID:" + Thread.currentThread().getId());
	executor. scheduleAtFixedRate(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task A. ThreadId:" + Thread.currentThread().getId());
		}
	}, 5, 3, TimeUnit.SECONDS);
}
```

### 出力

>08-29 17:59:33.556: D/ExecutorTest(7228): Primary ThreadID:5795<br/>
08-29 17:59:38.561: D/ExecutorTest(7228): Run task A. ThreadId:5796<br/>
08-29 17:59:41.565: D/ExecutorTest(7228): Run task A. ThreadId:5796<br/>
08-29 17:59:44.568: D/ExecutorTest(7228): Run task A. ThreadId:5796<br/>
…

最初は５秒、その後は３秒毎にタスクAが実行される。

## 7. 繰り返し実行されるタスクが重かったら？

繰り返しは３秒だけど、タスクAの実行に１０秒かかったら、どうなる？

```java
public void singleThreadScheduledExecutorTimerHeavyTest() throws Exception {
	final ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();

	Log.d(TAG, "Primary ThreadID:" + Thread.currentThread().getId());
	executor.scheduleAtFixedRate(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task A. ThreadId:" + Thread.currentThread().getId());
		}
	}, 5, 3, TimeUnit.SECONDS);
}
```

### 出力

>08-29 18:15:49.397: D/ExecutorTest(10157): Primary ThreadID:6188<br/>
08-29 18:15:54.413: D/ExecutorTest(10157): Run task A start. ThreadId:6189<br/>
08-29 18:16:04.403: D/ExecutorTest(10157): Run task A end.<br/>
08-29 18:16:04.403: D/ExecutorTest(10157): Run task A start. ThreadId:6189<br/>
08-29 18:16:14.404: D/ExecutorTest(10157): Run task A end.<br/>
08-29 18:16:14.404: D/ExecutorTest(10157): Run task A start. ThreadId:6189<br/>
08-29 18:16:24.405: D/ExecutorTest(10157): Run task A end.<br/>
…


３秒置きに設定しているが、タスクAが終わらないので、終わったら**すぐに**、次のタスクを実行する。

## 8. scheduleWithFixedDelay ではどうなる？

``scheduleAtFixedRate`` の代わりに ``scheduleWithFixedDelay`` にしてみた。

```java
public void singleThreadScheduledExecutorFixedDelayHeavyTest() throws Exception {
	final ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();

	Log.d(TAG, "Primary ThreadID:" + Thread.currentThread().getId());
	executor.scheduleWithFixedDelay(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task A. ThreadId:" + Thread.currentThread().getId());
		}
	}, 5, 3, TimeUnit.SECONDS);
}
```

### 出力

>08-29 18:22:37.183: D/ExecutorTest(10663): Primary ThreadID:6245<br/>
08-29 18:22:42.178: D/ExecutorTest(10663): Run task A start. ThreadId:6246<br/>
08-29 18:22:52.179: D/ExecutorTest(10663): Run task A end.<br/>
08-29 18:22:**55**.182: D/ExecutorTest(10663): Run task A start. ThreadId:6246<br/>
08-29 18:23:05.182: D/ExecutorTest(10663): Run task A end.<br/>
08-29 18:23:08.176: D/ExecutorTest(10663): Run task A start. ThreadId:6246<br/>
…


タスクAが終わって、**さらに３秒待って**、次のタスクを実行する。
``FixedDelay`` は、前回のタスクが終わってからｎ秒待つ。
``FixedRate`` は、の終了を待たずにｎ秒置きに実行するが、終わってない場合は仕方がないので終わるまで待つ、という感じらしい。

## 9. ScheduledThreadPoolExecutor ではどうか？

ScheduledThreadPoolExecutor  と ``scheduleAtFixedRate`` の組み合わせではどうか？

```java
public void singleThreadScheduledExecutorFixedDelayHeavyTest() throws Exception {
	final ScheduledExecutorService executor = Executors.newScheduledThreadPool(2);

	Log.d(TAG, "Primary ThreadID:" + Thread.currentThread().getId());
	executor.scheduleAtFixedRate(new Runnable() {
		@Override
		public void run() {
			Log.d(TAG, "Run task A start. ThreadId:" + Thread.currentThread().getId());
			
			// Wait
			try { Thread.sleep(10000); } 
			catch (InterruptedException e) { Log.w(TAG, "Interrupted task A. ThreadId:" + Thread.currentThread().getId()); } 
			
			Log.d(TAG, "Run task A end.");
		}
	}, 5, 3, TimeUnit.SECONDS);
}
```

### 出力

>08-29 19:09:39.707: D/ExecutorTest(14173): Primary ThreadID:6605<br/>
08-29 19:09:44.712: D/ExecutorTest(14173): Run task A start. ThreadId:6606<br/>
08-29 19:09:54.713: D/ExecutorTest(14173): Run task A end.<br/>
08-29 19:09:54.713: D/ExecutorTest(14173): Run task A start. ThreadId:6606<br/>
08-29 19:10:04.713: D/ExecutorTest(14173): Run task A end.<br/>
08-29 19:10:04.713: D/ExecutorTest(14173): Run task A start. ThreadId:6606<br/>
08-29 19:10:14.714: D/ExecutorTest(14173): Run task A end.<br/>
08-29 19:10:14.714: D/ExecutorTest(14173): Run task A start. ThreadId:6606<br/>
08-29 19:10:24.705: D/ExecutorTest(14173): Run task A end.<br/>
08-29 19:10:24.705: D/ExecutorTest(14173): Run task A start. ThreadId:6606<br/>
08-29 19:10:34.705: D/ExecutorTest(14173): Run task A end.<br/>
…

あれ？２つのスレッドを使ってくれない。を登録した時点でスレッドは決まってるということかな。

## まとめ

「タスクが実行されるスレッド」を意識すればハマることはなさそう。

シングルスレッドの場合は、``submit`` あるいは ``schedule`` されたタスクは、一つのスレッドで順次処理される。スレッドプールを使っている場合は、スレッドの数だけ並列処理される。
ただし、``scheduleAtFixedRate`` など繰り返し処理では、登録時にスレッドが決まるので、タスクの実行に時間がかかっても並列処理されない。

``scheduleAtFixedRate`` や ``scheduleWithFixedDelay`` はタイマー的な動きをするが、タスクの処理に時間がかかる場合は、意図した時間間隔で実行されない。タイマーとして使いたければ、Executor を２つ用意し、一つはタイマー専用、もうひとつをタスク実行専用とした方が良さそう。


長くなってしまったので、タスクのキャンセルとか、Terminate 系は別の機会に。