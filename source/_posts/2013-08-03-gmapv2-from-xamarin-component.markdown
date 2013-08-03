---
layout: post
title: "Xamarin.Android での Google Map(というか Play Service) 利用が、本家より簡単になった件"
date: 2013-08-03 15:29
comments: true
categories: [Xamarin, C#, Android, GoogleMapsAPI]
---
Xamarin Components に「Google Play Services」が追加されまして。
<!--more-->
* [Introducing the Google Play Services Component for Xamarin.Android | Xamarin Blog](http://blog.xamarin.com/introducing-the-google-play-services-component-for-xamarin-android/)

これが何を意味するかと言うと、これまで Google Play Service を利用するには、ライブラリプロジェクトを作って、アプリから参照するという煩わしい手順が必要でした。

で、これは Android-Eclipse でも同じく面倒だったわけですが、Xamarin.Android  に新しく提供されるこのコンポーネントを使えば、その手間を省くことができます。この点で Eclipse での開発より簡単になりました。

実際に、Play Services の一つである Google Map Android API v2 を使うアプリを作る手順を書いてみます。

## 手順

### 1. Google API Console から API key を取得する

この手順は、これまでと変わらないので、以下のサイトなどを参考にしてください。
package名が必要になるので、先に決めておきましょう。
ここでは ``com.amay077.sample.googlemapv2sample`` とします。

* [throw Life - Google Maps Android API v2を使ってみた](http://www.adamrocker.com/blog/334/google-maps-android-api-v2.html)

取得して API key はメモっておきます。

### 2. プロジェクト/ソリューションを作る

Xamarin Studio を起動します。
ここでは Ice Cream Sandwich 用に作ります。(Android Application の方だと Support Library が要るので少し手順が増えるはず)

![img](https://dl.dropboxusercontent.com/u/264530/qiita/gmapv2_from_xamarin_component_01.png)

### 3. プロジェクトに「Google Play Service」コンポーネントを追加する

メニュー → プロジェクト → Get More Components から、、、

![img](https://dl.dropboxusercontent.com/u/264530/qiita/gmapv2_from_xamarin_component_03.png)

Google Play Services を検索して Add to App します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/gmapv2_from_xamarin_component_04.png)

すると、プロジェクトに Google Play Services が追加されます。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/gmapv2_from_xamarin_component_05.png)

ここで一度、Xamarin Studio を再起動しておいた方が無難です。
このまま継続したら、追加されたアセンブリがうまく読み込まれてない場合がありました。

### 4. 実装する

``MainActivity.cs`` は、最初の内容をごっそり削除して以下のようにします。

```c# MainActivity.cs
using Android.App;
using Android.OS;

namespace GoogleMapV2Sample
{
    [Activity (Label = "GoogleMapV2Sample", MainLauncher = true)]
    public class MainActivity : Activity
    {
        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);

            // Set our view from the "main" layout resource
            SetContentView(Resource.Layout.Main);
        }
    }
}
```

画面定義である ``Main.axml`` も以下のように置き換えます。

```xml Resources/layout/Main.axml
<?xml version="1.0" encoding="utf-8"?>
<fragment xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/map"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    class="com.google.android.gms.maps.MapFragment" />
```

### 5. AndroidManifest.xml への設定いろいろ

たぶん一番面倒なところです。

まず Xamarin.Android では、最初は AndroidManifest.xml が生成されていないので、メニュー → プロジェクト → xxx のオプション から、下図のように [Add Android manifest] します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/gmapv2_from_xamarin_component_06.png)

次に作成された AndroidManifest.xml を開いて、以下のようにします。

※1 のところは、最初に決めた Package名に、※2 の時は、先に取得しておいた API key に置き換えてください。

```xml Properties/AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" 
	android:versionCode="1" 
	android:versionName="1.0" 
	package="com.amay077.sample.googlemapv2sample">  <--------※1


	<uses-feature android:glEsVersion="0x00020000" android:required="true" />
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="com.google.android.providers.gsf.permission.READ_GSERVICES" />
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
	<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

   
	<uses-permission 
		android:name="com.amay077.sample.playservicesample.permission.MAPS_RECEIVE" />   <--------※1
	<permission 
		android:name="com.amay077.sample.playservicesample.permission.MAPS_RECEIVE"   <--------※1
		android:protectionLevel="signature" />

	<application android:label="GoogleMapV2Sample">
		<meta-data 
			android:name="com.google.android.maps.v2.API_KEY" 
			android:value="AIzaByD1jiFER3le_HFrkOrEhaNIsemoNoDesuU" />  <--------※2
	</application>
</manifest>
```

### 6. 動かす

設定が正しくできていれば、地図が表示されるはずです。
アプリが落ちるなら Main.cs や Main.axml が、地図が表示されないなら AndroidManifest.xml や Google API Console での設定が間違っていると思います。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/gmapv2_from_xamarin_component_07.png)


## まとめなど

Google Play Service コンポーネントを使うことでライブラリプロジェクトをなくす事ができました。慣れた人にはどうってこと無い話ですが、説明する人には面倒で、始めて行う人には混乱の元になってたと思います。

もう一つ特筆すべきは、このコンポーネントを Google 自身が開発、提供していることです。

これだけでなく、[Google Map SDK for iOS](http://components.xamarin.com/view/googlemapsios/) や [Admob 用のコンポーネント](http://components.xamarin.com/view/googleadmob/)も Google 自身が提供しています。

また Microsoft も [Azure Mobile Service](http://components.xamarin.com/view/azure-mobile-services/) を自身が提供していますし、なんなんでしょうこのプラットフォーマーの Xamarin への参入ぶりは。

このように本家が開発していることにより、信頼性、機能網羅性、新機能への追従などがとても充実しており、安心して使うことができます。

最後に、

<blockquote class="twitter-tweet"><p>おおおお、すげえ！これでめんどいビルド手順必要なくなった！Google++！でも、Gmaps iOSの1.4アップデートと、Retina対応のため128dpタイルへの対応もしてください…。 / “Google Play Servi…” <a href="http://t.co/mjnN7sE0t7">http://t.co/mjnN7sE0t7</a></p>&mdash; Кочизуфан (@kochizufan) <a href="https://twitter.com/kochizufan/statuses/363116371351052290">August 2, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

とのことなのでよろしくおねがいします。