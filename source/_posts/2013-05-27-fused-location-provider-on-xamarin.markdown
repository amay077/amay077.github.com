---
layout: post
title: "Xamarin.Android で Fused Location Provider(など)を使う"
date: 2013-05-27 21:24
comments: true
categories: [Xamarin, Android, GoogleMapsAPI, C#]
---
Xamarin.Android は [Java ライブラリから C# のラッパを生成する機能](http://docs.xamarin.com/guides/android/advanced_topics/java_integration_overview/binding_a_java_library_(.jar) がとっても強力(Binding というみたい)なので、Fused Location Provider や Geofencing など、Google I/O 2013 で発表された新機能が入った google-play-service.jar も使えるはずだ、と思い試してみました。
<!--more-->
## monodroid-samples をベースに

Xamarin.Android のサンプル集 [monodroid-samples](https://github.com/xamarin/monodroid-samples) に、既に Google Map v2 を使うサンプルがあり、これが Google Play Service を使っているので、これを参考にします。

これね → [MapsAndLocationDemo_v2](https://github.com/xamarin/monodroid-samples/tree/master/MapsAndLocationDemo_v2) 、使い方は、

* [Xamarin.Android で Google Maps Android API v2 を使う - Experiments Never Fail](http://amay077.github.io/blog/2013/03/05/xamarin-android-using-google-maps-android-api-v2/)

をどうぞ。

## Binding の設定をいじる

Android SDK の Google Play Service をアップデートしても、Xamarin 側ですぐに ``LocationClient`` などが使えるわけではないです。
プロジェクト GooglePlayServices で、ラップする package などを設定しているため。

その設定は ``Transform/Metadata.xml`` にあるので、これを以下のように設定します。

```xml Metadata.xml
<metadata>
	<remove-node path="/api/package[@name='com.google.android.gms.maps']/class[@name='GoogleMapOptionsCreator']" />
	<remove-node path="/api/package[@name!='com.google.android.gms.maps' 
		and @name != 'com.google.android.gms.maps.model' 
		and @name != 'com.google.android.gms.common'
		and @name != 'com.google.android.gms.location']" />
	<remove-node path="/api/package[@name='com.google.android.gms.maps.model']/class[contains (@name, 'Creator')]" />
	<remove-node path="/api/package[@name='com.google.android.gms.location']/class[contains (@name, 'Creator')]" />
		
	<attr path="/api/package[@name='com.google.android.gms.maps']" name="managedName">Android.Gms.Maps</attr>
	<attr path="/api/package[@name='com.google.android.gms.maps.model']" name="managedName">Android.Gms.Maps.Model</attr>
	<attr path="/api/package[@name='com.google.android.gms.common']" name="managedName">Android.Gms.Common</attr>  
	<attr path="/api/package[@name='com.google.android.gms.location']" name="managedName">Android.Gms.Location</attr>  
	
  	<attr path="/api/package[@name='com.google.android.gms.maps.model']/class[@name='MarkerOptions']/method[@name='position']" name="managedName">SetPosition</attr>
	<attr path="/api/package[@name='com.google.android.gms.maps.model']/class[@name='MarkerOptions']/method[@name='snippet']" name="managedName">SetSnippet</attr>
	<attr path="/api/package[@name='com.google.android.gms.maps.model']/class[@name='MarkerOptions']/method[@name='title']" name="managedName">SetTitle</attr>
</metadata>
```

``com.google.android.gms.location`` を各所に追加しています。変更前は [こちら](https://github.com/xamarin/monodroid-samples/blob/master/MapsAndLocationDemo_v2/GooglePlayServices/Transforms/Metadata.xml) 。

これだけで OK。(がここにたどり着くまでに半日くらいかかったorz)

## 使う

これでリビルドすると、この GooglePlayServices を参照しているプロジェクトで、``Android.Gms.Location.LocationClient`` などが使えるようになります。（Xamarin Studio を再起動しないとコード入力補完(Inteli先生 というのか？) に認識されないみたいですが。）
