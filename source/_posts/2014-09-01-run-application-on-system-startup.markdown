---
layout: post
title: "システムの起動時にアプリを起動する"
date: 2014-09-01 21:41:43 +0900
comments: true
categories: [Android, Java]
---
ググれば出てくるんだけど、情報が古いので書きなおしてみた。
<!--more-->
## 全体

Android OS の起動が終わると ``android.intent.action.BOOT_COMPLETED`` がブロードキャストされるので、それを捕まえて任意の処理をする。


## 起動時に呼び出されるコード

ブロードキャストを捕まえたときに呼ばれるコード。``MyActivity`` を開始している。BroadcastReceiver から Activity を開始するには ``Intent.FLAG_ACTIVITY_NEW_TASK`` が必要なので注意。


```java StartupReceiver.java
public class StartupReceiver extends BroadcastReceiver {
    private static final String TAG = "StartupReceiver";

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d(TAG, "onReceive:" + MyApplication.data);
        Intent intentActivity = new Intent(context, MyActivity.class);
        intentActivity.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intentActivity);
    }
}
```

## AndroidManifest.xml で受信登録

``StartupReceiver`` を登録する。
忘れちゃいけないのが ``android.permission.RECEIVE_BOOT_COMPLETED`` による権限の設定。これがないと受信できない。

```xml AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.amay077.reboottest" >

    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".MyActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

        </activity>
        <receiver android:name=".StartupReceiver" >
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </receiver>
    </application>

</manifest>
```

## 端末を再起動して試す

* [AndroidのBOOT_COMPLETEDの受信とテスト | 9ensanのLifeHack](http://9ensan.com/blog/smartphone/android/android-boot_completed-adb-shell-am-broadcast/)

で知った ``adb shell am broadcast -a android.intent.action.BOOT_COMPLETED`` は GenyMotion でも使えました。
``RECEIVE_BOOT_COMPLETED`` の位置によっては、テストが成功したりしなかったりだと書かれておられますが、上記の ``AndroidManifest.xml`` では、テストも実際の再起動も成功しました。

## 参考

* [システムの起動時にサービスを実行する « Tech Booster](http://techbooster.jpn.org/andriod/application/1100/)(2010年なのでだいぶ古い、要注意)
* [AndroidのBOOT_COMPLETEDの受信とテスト | 9ensanのLifeHack](http://9ensan.com/blog/smartphone/android/android-boot_completed-adb-shell-am-broadcast/)(2012年、まだまだ古い)
* [broadcastreceiver - BOOT_COMPLETED not working Android - Stack Overflow](http://stackoverflow.com/questions/20441308/boot-completed-not-working-android/20441442#20441442)(2013年、これなら何とか)
* [BOOT_COMPLETEDが受信出来ない - Google グループ](https://groups.google.com/forum/#!topic/android-group-japan/D1EKohMIji0) SDカードにインストールされるとこのブロードキャストを受信できないそうです
