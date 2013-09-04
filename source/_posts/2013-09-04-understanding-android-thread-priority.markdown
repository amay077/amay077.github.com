---
layout: post
title: "スレッドの優先度について調べた"
date: 2013-09-04 22:22
comments: true
categories: [Android, Java]
---
Android にも(Java ですから)スレッドの優先度ってありますけど、それちゃんと動きますよね？というのを ExecurorService を使って調べた。
<!--more-->
## ThreadFactory の拡張

``ExecutorService`` が作るスレッドは、何もしないと 優先度:中 になる模様。
これを変更するには、生成時(``newSingleThreadExecutor``) に渡す ``ThreadFactory`` を自前で実装して、``Thread.setPriority`` してやる。

``ThreadFactory`` を Implements したクラス作ってもいいけど、そこまでやる必要も無いでしょ。

```java
// 指定した Priority の ThreadFactory を生成して返す
private ThreadFactory makeThreadFactory(final int priority) {
    return new ThreadFactory() {
        
        @Override
        public Thread newThread(Runnable r) {
            Thread thread = new Thread(r);
            thread.setPriority(priority);
            return thread;
        }
    };
}

// 使ってみる
private void testExecutor() {
    final ExecutorService executor = 
        Executors.newSingleThreadExecutor(makeThreadFactory(Thread.MIN_PRIORITY));

    executor.submit(new Runnable() {
        @Override
        public void run() {
            // なにかの処理
        }
    });
}
```

## 優先度が考慮されるかの検証

設定した Priority が正しく機能するのか試してみた。

### 検証方法

* MIN, NORMAL, MAX の優先度を設定をした、SingleThread な Executor を1つずつ、計３つ生成。
* ３つの Executor に、タスクをじゃんじゃん投入(submit)する。
* タスクの処理が開始された所をログ出力して、その順番を調べる。

### 検証コード

上を実装したのがこれ。``CountDownLatch`` で待ち合わせするのがなんだかなぁって感じだがまあいいや。

```java
private ThreadFactory makeThreadFactory(final int priority) {
    // 上と同じなので省略
}

// タスクを生成する
private Runnable makeTask(final String tag, final StringBuffer buffer, 
        final CountDownLatch latch) {
    return new Runnable() {
        @Override
        public void run() {
            // タスクが開始された時に、A or B or C を追加してく
            buffer.append(tag); // StringBuffer は Thread-safe なハズだ

            // Wait
            for (int j = 0; j < 1000000; j++) { } 

            latch.countDown();
        }
    };
}

// 検証実行
public void testThreadPriority() throws InterruptedException {
    // MIN, NORMAL, MAX な Priority の Executor を生成
    final ExecutorService minExecutor = 
            Executors.newSingleThreadExecutor(makeThreadFactory(Thread.MIN_PRIORITY));
    final ExecutorService norExecutor = 
            Executors.newSingleThreadExecutor(makeThreadFactory(Thread.NORM_PRIORITY));
    final ExecutorService maxExecutor = 
            Executors.newSingleThreadExecutor(makeThreadFactory(Thread.MAX_PRIORITY));

    final CountDownLatch latch = new CountDownLatch(100 * 3);
    final StringBuffer builder = new StringBuffer();
    
    // それぞれの Executor にじゃんじゃんタスクを投入
    for (int i = 0; i < 100; i++) {
        minExecutor.submit(makeTask("C", builder, latch)); // MIN->"C"
        norExecutor.submit(makeTask("B", builder, latch)); // NORMAL->"B"
        maxExecutor.submit(makeTask("A", builder, latch)); // MAX->"A"
    }
    
    // 全部のタスクが終わるのを待つ
    latch.await();
    Log.d(TAG, builder.toString()); // タスクが処理された順番を出力
}
```

### 検証結果(HTC J)

#### 1回目

> BAACAAABABABABABABABABABABABABABABABABACACACACBAABABABABABAB
ABABABABABABACBAABABABABABABABAABBABABABABABAABABABABABABABA
BABABABAABABABABABABABABABABABAABABABABABABABABABABABABABABA
BABABABABABABABCBBBBCBBCBCBBCBCBCCCCCCCCCCCCCCCCCCCCCCCCCCCC
CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC 

#### 2回目

> CBACABAABABABABABABAABABABABABABABABABACBABABABAABABBABABABA
ABABABABABABABABABABABABABABABABABABABABABABABABABABAABABABA
BABABABABBABAABABABABABABABABABABABABABABABABABCABABABABABAB
ABABABABABABAABABABBCBCBBCBCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC

#### 3回目

> BAACAABABABAABABABABABABABABABABABABABABABABABABABABABABABAB
ABABABABABABABABABABABABABABABABABABABABABABABABABABABABABAB
ABABABABABABABABABABABABABACBABABABABABABABABABABABABABABABA
BABABABAABABABABABCBBCBCBCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC

"C"(優先度:MIN) のタスクが後ろの方に追いやられているのが分かる。
"A"(MAX) と "B"(NORMAL) の差ははっきりとはわからないが、それでも "A" の方が先に全部終わっているので優先されているのがなんとか分かる。

## まとめ

優先度の変更、ちゃんと効果あるんですね。

よく調べると Android には ``android.os.Process.setThreadPriority()`` というのもあるそうで。

* [android.os.Process.setThreadPriority()とjava.lang.Thread.setPriority()の効果は同じ - みかん箱](http://mikanbako.blog.shinobi.jp/android/android.os.process.setthreadpriority--%E3%81%A8java.lang.thread.setpriority--%E3%81%AE%E5%8A%B9%E6%9E%9C%E3%81%AF%E5%90%8C%E3%81%98)

> 結果、両者の効果は同じです。ただし、android.os.Processではスレッドの優先度が用途ごとに定数として定義されているため、まずandroid.os.Process.setThreadPriority()の使用を検討すべきでしょう。

とのこと。確かに ``THREAD_PRIORITY_FOREGROUND``, ``THREAD_PRIORITY_AUDIO``, ``THREAD_PRIORITY_LOWEST`` などが定義されているので、こちらを使った方が良さげですね。
個人的には、Java の API でできる事はその範囲で閉じてしまいたいのですがね。