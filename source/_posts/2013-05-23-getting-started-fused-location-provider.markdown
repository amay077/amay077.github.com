---
layout: post
title: "Google I/O 2013 で発表された Fused Location Provider を使ってみる"
date: 2013-05-23 20:55
comments: true
categories: [Android, GoogleMapsAPI]
---
[Activity Recognition](http://amay077.github.io/blog/2013/05/18/getting-started-activity-recognition/) に続いて使ってみました。
<!--more-->
## Fused Location Provider とは？

GPS と WiFi とセンサー(加速度など) を組み合わせて、その状況に応じた最適な方法で位置を取得出来ます。今までよりも低消費電力で、精度のよい位置情報を。

実際どんな感じかは、Google I/O 2013 のセッション動画にある [Fused Location Provider のデモ](http://www.youtube.com/watch?feature=player_detailpage&v=URcVZybzMUI#t=733s) を見てください。

次から使い方です。

## 1. SDK の Google Play Services を更新する

![image1](https://dl.dropboxusercontent.com/u/264530/qiita/getting_started_activity_recognition1.png)

Activity Recognition と同じく Google Play services として提供されているので、SDK Manager でライブラリを更新します。

## 2. Eclipse でプロジェクトを作る

Android Studio は使ってません(まだよくわからないので)
Eclipse で、いつもどおりに Android のプロジェクトを作ります。
Fragment も使いませんよ、古きよき、BlankActivity なプロジェクトです。
名前はここでは FusedLocationProviderSample とします。

## 3. プロジェクトに google-play-services_lib を追加する

Google Play services を使うために、SDK のフォルダにある google-play-services_lib が必要です。

Ecplise の Import で ``{your sdk location}/extras/google/google_play_services/libproject/google-play-services_lib`` を選択します。自分の Workspace に Copy しておいた方が無難でしょう。

コピーしたら、FusedLocationProviderSample で、 google-play-services_lib をライブラリ参照します。

![image2](https://dl.dropboxusercontent.com/u/264530/qiita/getting_started_activity_recognition2.png)

次から FusedLocationProviderSample の実装です。

## 4. AndroidManifest.xml の編集

Fused Location Provider を使うのには今まで通り、``ACCESS_FINE_LOCATION`` と ``ACCESS_COARSE_LOCATION`` を設定します。

LocationClient は、指定した PERMISSION に応じてよしなに動いてくれるそうです。

```xml AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.fusedlocationprovidersample"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <activity
            android:name="com.example.fusedlocationprovidersample.MainActivity"
            android:screenOrientation="portrait"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

## 5. Fused Location Provider を使う

ActivityRecognition は IntentService を使う必要があったために少々長いコードになりましたが、こちらは今までの ``LocationProvider`` が(も)使えるので、画面一つだけのシンプルなコードです。

FusedLocationProvider を使うには、[LocationClient](http://developer.android.com/reference/com/google/android/gms/location/LocationClient.html) クラスを使います。

大雑把な流れは:

1. インスタンスを生成する
connect を呼ぶ -> ConnectionCallbacks.onConnected がコールバックされる
2. [LocationRequest](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html) を指定して、requestLocationUpdates を呼ぶ。 -> LocationListener.onLocationChanged が呼ばれる

です。connect を呼んで onConnected を待つのと、位置取得条件が ``LocationRequest`` クラスになった以外は ``LocationManager`` と同じです。

では、全コードです。

```java MainActivity.java
package com.example.fusedlocationprovidersample;

import com.google.android.gms.common.ConnectionResult;
import com.google.android.gms.common.GooglePlayServicesClient.ConnectionCallbacks;
import com.google.android.gms.common.GooglePlayServicesClient.OnConnectionFailedListener;
import com.google.android.gms.location.LocationClient;
import com.google.android.gms.location.LocationListener;
import com.google.android.gms.location.LocationRequest;
import android.location.Location;
import android.os.Bundle;
import android.text.format.DateFormat;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.TextView;
import android.app.Activity;

public class MainActivity extends Activity {
	
	// FusedLocationProvider 用の Client
	private LocationClient _locationClient;
	private TextView _textResult;
	
	// 以前と変わらない LocationListener
    private final LocationListener _locationListener = new LocationListener() {
		
		@Override
		public void onLocationChanged(final Location location) {
			MainActivity.this.runOnUiThread(new Runnable() {
				@Override
				public void run() {
					String text = _textResult.getText().toString();
		            text = DateFormat.format("hh:mm:ss.sss", location.getTime()) + " - " 
		                    + location.getLatitude() + "/" +
		                    + location.getLongitude() + "/" +
		                    + location.getAccuracy() + 
		                    "\n" + text;

		            _textResult.setText(text);
				}
			});
		}
	};
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		_textResult = (TextView)findViewById(R.id.text_result);
		
		final Button buttonLocate = (Button)findViewById(R.id.button_locate);
		buttonLocate.setOnClickListener(new OnClickListener() {
			private boolean _isStarted = false;
			
			@Override
			public void onClick(View v) {
				if (!_isStarted) {
					startLocate();
					buttonLocate.setText("Stop");
				} else {
					stopLocate();
					buttonLocate.setText("Start");
				}
				
				_isStarted = !_isStarted;
			}
		});
	}
	
	@Override
	protected void onDestroy() {
		stopLocate();
		super.onDestroy();
	}
	
	private void startLocate() {
		_locationClient = new LocationClient(this, new ConnectionCallbacks() {

			@Override
            public void onConnected(Bundle bundle) {
				// 2. 位置の取得開始！
				LocationRequest request = LocationRequest.create()
				.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY)
				.setInterval(5000); // 5秒おき
            	_locationClient.requestLocationUpdates(request, _locationListener);
            }

            @Override
            public void onDisconnected() {
            	_locationClient = null;
            }

        }, new OnConnectionFailedListener() {
            @Override
            public void onConnectionFailed(ConnectionResult result) {
            }
        });
		
		// 1. 位置取得サービスに接続！
		_locationClient.connect();
	}

	private void stopLocate() {
		if (_locationClient == null || !_locationClient.isConnected()) {
			return;
		}
		
		_locationClient.removeLocationUpdates(_locationListener);
		_locationClient.disconnect();
		// ConnectionCallbacks.onDisconnected が呼ばれるまで待った方がいい気がする
	}
}
```

## 6. 動くのか！？

HTC J(not蝶) で動かしてみました。

![image3](https://dl.dropboxusercontent.com/u/264530/qiita/getting_started_fused_location_provider1.png)

室内での結果ですが、最初 27m の精度だったのが、放っておくとどんどん精度が上がって行きました。が、地図に重ねてみないと実際合ってるのかよくわかりませんね。

そのうち、地図に載せて検証してみたいです。

## Permission と Priority と精度の話

Permission と Priority の組み合わせで、位置の精度がどう変わるか、少し調べました。

### FINE_LOCATION+COARSE_LOCATION with PRIORITY_HIGH_ACCURACY

GPS と WiFi と センサーフル活用。GPS が捕捉できなくても数十ｍの位置精度が概ね出るようです。

### COARSE_LOCATION with PRIORITY_HIGH_ACCURACY

使えない。FINE_LOCATION が必要ってエラーになりました。

### COARSE_LOCATION with  PRIORITY_BALANCED_POWER_ACCURACY 

位置の精度が数km程度になりました。WiFi測位(従来の NETWORK_PROVIDER)よりも悪いです。うーんこれは期待はずれだなあ。

結局、例えば屋内測位でしか使わないからGPS要らねって FINE_LOCATION を外すと、かえって精度が落ちるという事になります。(GPS使いませんPERMISSIONが欲しいな。。。)

## おまけ

* [Google Maps Android API v2 Release Notes - Google Maps Android API v2 — Google Developers](https://developers.google.com/maps/documentation/android/releases#may_2013)

によると、Google Maps Android API v2 の [setMyLocationEnabled(true)](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/GoogleMap#setMyLocationEnabled(boolean) でも FusedLocationProvider が使われるようになったとのことです。


## まとめ

公式のコンプリートな Getting Started は

* [Retrieving the Current Location | Android Developers](http://developer.android.com/training/location/retrieve-current.html)

にありますので、こちらを読まれた方が確実です。

ここで作ったサンプルは、

* [amay077/fusedlocationprovidersample · GitHub](https://github.com/amay077/fusedlocationprovidersample)

に置いておきます。