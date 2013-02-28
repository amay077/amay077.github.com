---
layout: post
title: "jUnit で非同期処理のテストがちょっと楽になるクラス作ってみた"
date: 2012-12-20 19:07
comments: true
categories: [jUnit, Java]
---
非同期メソッドのテストって、皆さんどうやってるんでしょう？
ちょっとググってみたけど「```Object.wait``` とか ```CountDownLatch``` とか ```Future``` で待て」とかあんまり良い答えが見つからなかったので、自分でユーティリティクラス作ってみた。

<!-- more -->

まあ CountDownLatch で待ってるだけなんですけども。

2013.1.10 修正:メソッドに全部 ``synchronized`` つけたら動かんやん、恥ずかし…

```java FutureResult.java
/**
 * success または error が呼ばれるまで get() で待ってる Future みたいなクラス
 * 
 * @author @amay077
 */
public class FutureResult<T> {
	private final int TIMEOUT = 10;
	private final TimeUnit TIMEOUT_UNIT = TimeUnit.SECONDS;
	
	private final CountDownLatch _latch = new CountDownLatch(1);
	private T _value;
	private Exception _error;

	public static class FutureResultException extends Exception {
		private static final long serialVersionUID = 1L;

		public FutureResultException(Exception detailException) {
			super(detailException);
		}
	}
	
	/**
	 * 非同期処理が成功したら呼ぶメソッド
	 */
	public synchronized void success(T value) {
		_value = value;
		_latch.countDown();
	}

	/**
	 * 非同期処理が失敗したら呼ぶメソッド
	 */
	public synchronized void error(Exception ex) {
		_error = ex;
		_latch.countDown();
	}

	/**
	 * 非同期処理が終わるまで待って結果を返す。
	 * エラーだったら例外を投げる。
	 */
	public T get() throws Exception {
		try {
			if (!_latch.await(TIMEOUT, TIMEOUT_UNIT)) {
				throw new FutureResultException(new TimeoutException());
			}
		} catch (Exception ex) {
			throw new FutureResultException(ex);
		}
		
		if (_error != null) {
			throw _error;
		}
		
		return _value;
	}

}
```

Future インターフェースを implements しようと思ったけど数が多くてやめたｗ
使い方はこんな感じ。

```java AsyncMethodTest.java
public void testAsyncMethod() {
	final FutureResult<Integer> result = new FutureResult<Integer>();
	
	// 非同期なメソッドを実行
	hoge.asyncMethod(new OnReceiveListener() {
		@Override
		public void onReceive(Integer data) {
			// 正そうな値を受信しtら success を呼ぶ
			result.success(data);
		}
		
		@Override
		public void onError(Exception ex) {
			// エラーを受信した場合は error を呼ぶ
			result.error(ex);
		}
	});
	
	// 検証
	try {
		// get で success か error かタイムアウトするまで待ってる。
		Assert.assertEquals(0, result.get()); 
	} catch (Exception e) {
		fail(e.getMessage());
	}
}

```

## 参考

なんて記事を書いたあとにもっかいググってみたらこんなライブラリがあるようで。詳細はまだ見てない。

* [Awaitility](http://code.google.com/p/awaitility/) - Awaitility is a small Java-based DSL for synchronizing asynchronous operations. It makes it easy to test asynchronous code. - Google Project Hosting