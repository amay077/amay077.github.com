---
layout: post
title: "Google Maps SDK(Android/iOS) で円を描く"
date: 2013-05-06 16:11
comments: true
categories: [Xamarin, Android, iOS, GoogleMapsAPI]
---
[Google Maps Android API v2](https://developers.google.com/maps/documentation/android/) にいつの間にか ``Circle`` が追加されてました。[Google Maps SDK for iOS](https://developers.google.com/maps/documentation/ios/?hl=ja) も同様に。
<!--more-->
ということで使ってみました。なぜか Xamarin でｗ
とはいえ、Xamarin.Android の場合、GoogleMaps の jar からラッパーを生成しているので API は Java 版と(ほぼ)同じです。

Xamarin.Android での Google Maps API v2 の使い方は、手前味噌ながら弊ブログをどうぞ。

* [Xamarin.Android で Google Maps Android API v2 を使う - Experiments Never Fail](http://amay077.github.com/blog/2013/03/05/xamarin-android-using-google-maps-android-api-v2/)


今回も、[monodroid-samples](https://github.com/xamarin/monodroid-samples/tree/master/MapsAndLocationDemo_v2/SimpleMapDemo) をベースにします。

SampleMapActivity.cs の、[ここら辺](https://github.com/xamarin/monodroid-samples/blob/master/MapsAndLocationDemo_v2/SimpleMapDemo/SampleMapActivity.cs#L88) に以下のコードを挿入します。

```c# SampleMapActivity.cs
var ICELAND = new LatLng(64.88, -18.32);
var LIBREVILLE = new LatLng(0.401, 9.459);

_map.AddCircle(new CircleOptions()
               .InvokeCenter(ICELAND)
               .InvokeStrokeColor(Color.Blue.ToArgb())
               .InvokeStrokeWidth(5f)
               .InvokeRadius(5000000)); // 500km

_map.AddCircle(new CircleOptions()
               .InvokeCenter(LIBREVILLE)
               .InvokeStrokeColor(Color.DarkGreen.ToArgb())
               .InvokeStrokeWidth(5f)
               .InvokeRadius(500000)); // 500km
```

Xamarin.Android と Java で違いは無いと言っておきながら、
Java では ``new CircleOptions().center(…).storokeColor(…).`` と書いていた所が、Xamarin では ``new CircleOptions().InvokeCenter(…).InvokeStorokeColor(…).`` になっていますね。[Xamarin の Binding a Java Library (.jar)](http://docs.xamarin.com/guides/android/advanced_topics/java_integration_overview/binding_a_java_library_(.jar) 辺りに情報があるでしょうか。

動かすとこんな感じになります。

![image1](https://dl.dropboxusercontent.com/u/264530/qiita/circle_on_google_maps_sdk.png)

北緯65度付近のアイスランドと、赤道付近のリーブルヴィルという所に、どちらも半径500kmの円を追加していますが、Googleマップはメルカトル表示なので、見ての通り北へ行くほど円が大きくなります。

面白いのは「真円は保たれている事」ですね。逆台形っぽい曲線になるかと思ったのですが。

円をポリゴナイズした多角形と重ねてみると違いが分かるかもしれません。

### 2013.5.8追記:大きな円を描いて調べてみました

大きな円を描いてみたら、ちゃんと geodesic な(北へいけばいくほど距離が長くなる)形状になりました。

![image2](https://dl.dropboxusercontent.com/u/264530/qiita/circle-on-google-maps-sdk2.png)
