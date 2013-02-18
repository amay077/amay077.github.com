---
layout: post
title: "Android の jUnit テストで Context が欲しい時"
date: 2013-01-19 00:44
comments: true
categories: [Android, jUnit]
---
Activity のテストなら ``ActivityInstrumentationTestCase2`` 、 Service なら ```ServiceTestCase``` を使うべきなんでしょうけど、Android って事あるごとに Context が必要なので、ただのクラスライブラリのテストでも必要なことがシバシバ。

``` java HogeTest.java
public class HogeTest extends InstrumentationTestCase {
	/** ApplicationContext を取得します */
	private Context getApplicationContext() {
		return this.getInstrumentation().getTargetContext().getApplicationContext();
	}
}
```

## 参考
* [androidの単体テスト(AndroidTestCase) - Androidのド肝] (http://blog.haw.co.jp/android/?p=471) - クラス図がとても役に立ちました
* [android - Accessing application context from TestSuite in Setup() before calling getActivity() - Stack Overflow](http://stackoverflow.com/questions/5544205/accessing-application-context-from-testsuite-in-setup-before-calling-getactivi)