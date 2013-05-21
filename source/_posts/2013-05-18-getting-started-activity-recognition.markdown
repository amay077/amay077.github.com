---
layout: post
title: "Google I/O 2013 で発表された行動認識(Activity Recognition)を使ってみる"
date: 2013-05-18 11:37
comments: true
categories: [Android]
---
[Google I/O で発表](http://www.gizmodo.jp/2013/05/android_52.html)された Android の行動認識(動作認識)機能ですが、これは [Google Play Services](http://developer.android.com/google/play-services/index.html) で提供されているので、新しい API Ver でなくても(Froyo でも！)使えます、すばらしい！

というわけで、早速使ってみました。
<!--more-->
## 1. SDK の Google Play Services を更新する

![image1](https://dl.dropboxusercontent.com/u/264530/qiita/getting_started_activity_recognition1.png)

SDK Manager で、Google Play service を最新に更新します。
私は勢いで Android SDK Tools なども最新にしてしまいましたが、これが必要だったかは定かでないです。また SDK Tools を更新したら Eclipse のプラグインも更新する必要がありました。

## 2. Eclipse でプロジェクトを作る

Android Studio は使ってません(まだよくわからないので)
Eclipse で、いつもどおりに Android のプロジェクトを作ります。
Fragment も使いませんよ、古きよき、BlankActivity なプロジェクトです。
名前はここでは ``ActivityRecognizingSample`` とします。

## 3. プロジェクトに google-play-services_lib を追加する

Google Play services を使うために、SDK のフォルダにある google-play-services_lib が必要です。

Ecplise の Import で ``{your sdk location}/extras/google/google_play_services/libproject/google-play-services_lib`` を選択します。自分の Workspace に Copy しておいた方が無難でしょう。

コピーしたら、ActivityRecognizingSample で、 google-play-services_lib をライブラリ参照します。

![image2](https://dl.dropboxusercontent.com/u/264530/qiita/getting_started_activity_recognition2.png)

次から ActivityRecognizingSample の実装です。

## 4. IntentService クラスの用意

行動認識結果は IntentService で受け取ります。そのためのクラス ``ReceiveRecognitionIntentService`` を作成します。

```java ReceiveRecognitionIntentService.java
package com.example.activityrecognizingsample;

import android.app.IntentService;

public class ReceiveRecognitionIntentService extends IntentService {

	public ReceiveRecognitionIntentService()  {
		super("ReceiveRecognitionIntentService");
	}

	@Override
	protected void onHandleIntent(Intent intent) {
        // まだ未実装
	}
}
```


## 5. AndroidManifest.xml の編集

行動認識を使うための権限 ``com.google.android.gms.permission.ACTIVITY_RECOGNITION`` をマニフェストに追加します。
あと、ReceiveRecognitionIntentService も忘れずに追加しておきます。
細かいところでは、MainActivity の画面の向きを縦（Portrait）に固定しておきます。試している時に画面の向きが変わると面倒なので。

```xml AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.activityrecognizingsample"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />
    <uses-permission android:name="com.google.android.gms.permission.ACTIVITY_RECOGNITION"/>

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name" 
            android:screenOrientation="portrait">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <service 
            android:name=".ReceiveRecognitionIntentService"   
            android:label="@string/app_name"
    		android:exported="false" />
    </application>

</manifest>
```

## 6. 行動認識クラス(ActivityRecognitionClient)を使う

ここから、一気に行きます。

行動認識には [``ActivityRecognitionClient``](http://developer.android.com/reference/com/google/android/gms/location/ActivityRecognitionClient.html) を使います。

大雑把な使い方は、

1. インスタンスを生成する
2. ``connect`` を呼ぶ -> ``ConnectionCallbacks.onConnected`` がコールバックされる
3. ``requestActivityUpdates`` に ``ReceiveRecognitionIntentService`` を仕掛けて、呼ぶ。 -> ``ReceiveRecognitionIntentService.onHandleIntent`` が呼ばれる
4. ``onHandleIntent`` で ``ActivityRecognitionResult`` にて結果を取得する

です。

その後、認識結果を画面に表示するために、

5. 予め、``MainActivity`` に ``BroadcastReceiver`` を仕掛けておく
6. ``ReceiveRecognitionIntentService.onHandleIntent`` で取得した認識結果を、Broadcast する。

とします。

では全コードをどうぞ。

```java MainActivity.jara
package com.example.activityrecognizingsample;

import com.google.android.gms.common.ConnectionResult;
import com.google.android.gms.common.GooglePlayServicesClient.ConnectionCallbacks;
import com.google.android.gms.common.GooglePlayServicesClient.OnConnectionFailedListener;
import com.google.android.gms.location.ActivityRecognitionClient;
import com.google.android.gms.location.DetectedActivity;

import android.os.Bundle;
import android.app.Activity;
import android.app.PendingIntent;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.text.format.DateFormat;
import android.view.Menu;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends Activity {
    private ActivityRecognitionClient _recClient; // 行動認識のメインクラス
	private TextView _textResult; // 認識結果を表示するところ
	
	// 認識結果は PendingIntent で通知してくれる
	//  PendingIntent に、Service を起動する Intent を仕込んでおいて、
	//  認識結果の取得はそっちで行う。 > ReceiveRecognitionIntentService.java
    private PendingIntent _receiveRecognitionIntent; 
    
    // ReceiveRecognitionIntentService で取得した認識結果は、Broadcast で通知されるので、
    // それを受け取る Receiver 。ここで画面に認識結果を表示する。
	private final BroadcastReceiver _receiveFromIntentService = new BroadcastReceiver() {
		@Override
		public void onReceive(Context context, Intent intent) {
			final int activityType = intent.getIntExtra("activity_type", 0);
			final int confidence = intent.getIntExtra("confidence", -1);
			final long time = intent.getLongExtra("time", 0);
			
			MainActivity.this.runOnUiThread(new Runnable() {
				@Override
				public void run() {
					String text = _textResult.getText().toString();
					text = DateFormat.format("hh:mm:ss.sss", time) + " - " 
							+ getNameFromType(activityType) + "(" +
							+ confidence + ")" + "\n" + text;
					
					_textResult.setText(text);
				}
			});
		}
		
		// http://developer.android.com/training/location/activity-recognition.html
		// からパクってきた関数
		 /**
	     * Map detected activity types to strings
	     *@param activityType The detected activity type
	     *@return A user-readable name for the type
	     */
	    private String getNameFromType(int activityType) {
	        switch(activityType) {
	            case DetectedActivity.IN_VEHICLE:
	                return "in_vehicle";
	            case DetectedActivity.ON_BICYCLE:
	                return "on_bicycle";
	            case DetectedActivity.ON_FOOT:
	                return "on_foot";
	            case DetectedActivity.STILL:
	                return "still";
	            case DetectedActivity.UNKNOWN:
	                return "unknown";
	            case DetectedActivity.TILTING:
	                return "tilting";
	        }
	        return "unknown - " + activityType;
	    }
	};
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        _textResult = (TextView)findViewById(R.id.text_results);
        
        // IntentService から Broadcast される認識結果を受け取るための Receiver を登録しておく
        registerReceiver(_receiveFromIntentService, new IntentFilter("receive_recognition"));
        
        final Button buttonStart = (Button)findViewById(R.id.button_start);
        buttonStart.setOnClickListener(new OnClickListener() {
        	private boolean _isStarted = false;
        	
			@Override
			public void onClick(View v) {
				
				if (!_isStarted) {
					startReckoning();
					buttonStart.setText("Stop");
				} else {
					stopReckoning();
					buttonStart.setText("Start");
				}
				
				_isStarted = !_isStarted;
			}
		});
    }
    
    @Override
    protected void onDestroy() {
    	stopReckoning();
		// ConnectionCallbacks.onDisconnected が呼ばれるまで待った方がいい気がする
    	unregisterReceiver(_receiveFromIntentService);
    	super.onDestroy();
    }
    
    private void startReckoning() {
    	_recClient = new ActivityRecognitionClient(this, new ConnectionCallbacks() {
			
			@Override
			public void onConnected(Bundle bundle) {
				Intent intent = new Intent(
		                MainActivity.this, ReceiveRecognitionIntentService.class);
				_receiveRecognitionIntent = PendingIntent.getService(
						MainActivity.this, 0, intent,
		                PendingIntent.FLAG_UPDATE_CURRENT);
				
				// 2. 行動認識開始！
				//  1秒間隔で認識間隔を通知。
				//  認識したら ReceiveRecognitionIntentService が呼び出されるようにしている。
				_recClient.requestActivityUpdates(1000, _receiveRecognitionIntent);
			}

			@Override
			public void onDisconnected() {
				_recClient = null; // NOTE disconnect してもここにこないよ？
			}

    	}, new OnConnectionFailedListener() {
			@Override
			public void onConnectionFailed(ConnectionResult result) {
				// 接続でエラーが発生したらここにくるらしい
			}
		});
    	
    	// 1. 行動認識サービスに接続！
    	_recClient.connect();
    }
    
    private void stopReckoning() {
    	if (_recClient == null || !_recClient.isConnected()) {
    		return;
    	}
    	
    	_recClient.removeActivityUpdates(_receiveRecognitionIntent);
		_recClient.disconnect(); 
		// ConnectionCallbacks.onDisconnected が呼ばれるまで待った方がいい気がする
    }
    
    
}
```

```java ReceiveRecognitionIntentService.java
package com.example.activityrecognizingsample;

import com.google.android.gms.location.ActivityRecognitionResult;
import com.google.android.gms.location.DetectedActivity;

import android.app.IntentService;
import android.content.Intent;
import android.text.format.DateFormat;
import android.util.Log;

/**
 * 行動認識結果を取得するための IntentService 
 * 
 * ActivityRecognitionClient.requestActivityUpdates に仕込んでおくと
 * 認識結果を受信する度にこれが呼ばれる。
 * 
 */
public class ReceiveRecognitionIntentService extends IntentService {
	private static final String TAG = "ReceiveRecognitionIntentService";

	public ReceiveRecognitionIntentService()  {
		super("ReceiveRecognitionIntentService");
	}

	@Override
	protected void onHandleIntent(Intent intent) {
		if (!ActivityRecognitionResult.hasResult(intent)) {
			// 行動認識結果持ってないよ
			return;
		}
		
		// 認識結果を取得する
		ActivityRecognitionResult result = 
				ActivityRecognitionResult.extractResult(intent);
		
		DetectedActivity mostProbableActivity = result.getMostProbableActivity();
		int activityType = mostProbableActivity.getType();
		int confidence = mostProbableActivity.getConfidence();
		
		Log.d(TAG, "Receive recognition.");
		Log.d(TAG, " activityType - " + activityType); // 行動タイプ
		Log.d(TAG, " confidence - " + confidence); // 確実性（精度みたいな）
		Log.d(TAG, " time - " + DateFormat.format("hh:mm:ss.sss", result.getTime())); // 時間
		Log.d(TAG, " elapsedTime - " + DateFormat.format("hh:mm:ss.sss", result.getElapsedRealtimeMillis())); // よく分からん

		// 画面に結果を表示するために、Broadcast で通知。
		//  MainActivity にしかけた BroadcastReceiver で受信する。
		Intent notifyIntent = new Intent("receive_recognition");
		notifyIntent.setPackage(getPackageName());
		notifyIntent.putExtra("activity_type", activityType);
		notifyIntent.putExtra("confidence", confidence);
		notifyIntent.putExtra("time", result.getTime());
		sendBroadcast(notifyIntent);
	}
}
```

## 7. 動くのか！？

HTC J(not蝶) で動かしてみました。

![image3](https://dl.dropboxusercontent.com/u/264530/qiita/getting_started_activity_recognition3.jpg)

on_foot ってのが「歩いてる」ってやつですね。
感覚としてはズボンの尻ポケに入れて、5mくらいは歩かないと認識されない感じ。結構 unknown が多いですね。

クルマは、ダッシュボードに放置して運転してみたもの。開始から１分経たずに認識できています。

自転車は、最初クルマと誤認したものの、その後認識しました。

ここでは教科書通りの使い方をしましたが、なかなか感動します。

### 2013.5.21追記 電車だとどうなる？

東京に出張する機会があったので、新幹線と在来線でも試してみました。

![image4](https://dl.dropboxusercontent.com/u/264530/qiita/getting_started_activity_recognition4.png)

新幹線は窓側の席だったので、窓の机に端末を放置していました。
在来線は座ることができなかったので、立った状態で端末は尻ポケ、カベに持たれてなるべく動かないようにしていました。

結果は、在来線ではなんとか vehicle と認識されましたが、新幹線では認識できませんでした。
恐らく、新幹線は揺れが少なすぎるのだと思います。優秀ですね、日本の新幹線。

## まとめ

公式のコンプリートな Getting Started は

* [Recognizing the User's Current Activity | Android Developers](http://developer.android.com/training/location/activity-recognition.html)

にありますので、こちらを読まれた方が確実です。

ここで作ったサンプルは、

* [amay077/androidactivityrecognizingsample · GitHub](https://github.com/amay077/androidactivityrecognizingsample)

に置いておきます。