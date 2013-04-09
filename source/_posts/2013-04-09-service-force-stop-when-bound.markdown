---
layout: post
title: "bind されてても stopService で止まるサービスの作り方"
date: 2013-04-09 22:24
comments: true
categories: [Android]
---
Android のサービスは、``startService`` で「開始状態」、``bindService`` で「接続状態」となり、サービスを停止させる ``stopService`` は、``unbindService`` で接続を解除してから実行しないと止まらない、とドキュメントに書いてあります。
<!--more-->
[Managing the Lifecycle of a Service | Android Developers](http://developer.android.com/guide/components/services.html#Lifecycle) より

> In cases like this, stopService() or stopSelf() does not actually stop the service until all clients unbind.

それから、一見、サービスが切断された時に呼び出されるように見える [``ServiceConnection.onServiceDisconnected``](http://developer.android.com/reference/android/content/ServiceConnection.html#onServiceDisconnected(android.content.ComponentName) は、

* [Androidアプリ開発メモ032:サービス その4:バインドされたサービス: ぷ～ろぐ](http://into.cocolog-nifty.com/pulog/2011/10/android032-4-aa.html)
* [Serviceのライフサイクルの動作確認 - DenkiYagi](http://terurou.hateblo.jp/entry/20100519/1274252852)
* [Binding to a Service - Bound Services | Android Developers](http://developer.android.com/guide/components/bound-services.html#Binding)

> onServiceDisconnected()
The Android system calls this when the connection to the service is unexpectedly lost, such as when the service has crashed or has been killed. This is **not** called when the client unbinds.

とあるように、実はサービスが異常終了した時などしか呼ばれないとのことです。

今回、偶然にも、

* 接続状態なのに stopService でサービスが止まる
* ``ServiceConnection.onServiceDisconnected`` が呼ばれる

という動きをするサービスができたので、備忘録として残しておきます。

条件は、startService した後、bindService の第2引数を ``Context.BIND_AUTO_CREATE`` ではなく ``0`` にすること。

```java StartService.java
Intent intent = new Intent(context, TestService.class);

final ServiceConnection conn = new ServiceConnection() {
	public void onServiceConnected(ComponentName name, IBinder service) {
		Log.i("ServiceTest", "onServiceConnected");
			binder = ITestService.Stub.asInterface(service);
	}

	// stopService すると呼ばれる
	public void onServiceDisconnected(ComponentName name) {
		Log.i("ServiceTest", "onServiceDisconnected");
		context.unbindService(conn); // これ呼ばないとリークする
	}
};

// サービス開始＆接続
context.startService(intent);
context.bindService(intent, conn, 0); // not BIND_AUTO_CREATE


// ボタンを押したらサービス停止
findViewById(R.id.btnCallTerminate)
	.setOnClickListener(new OnClickListener() {
	public void onClick(View v) {
		context.stopService(intent);
	}
});
```

サービス側のコードは省略してしまったけど、stopService すると、

1. ServiceConnection.onServiceDisconnected
2. Service.onUnbind
3. Service.onDestroy

の順で呼ばれてます。onServiceDisconnected で ``context.unbindService`` を呼ばないと、アプリ終了時にメモリリークが検出されます。また、呼ばなくても、2. 3. は変わりません。

外から stopService でなく、サービス内から ``stopSelf`` でも同じでした。

接続中にサービスを停止しても、onServiceDisconnected で補足できるので、自然と言えば自然ですが、説明と違う動きをされると混乱しますね。(どこかに書いてあるのかな？)