---
layout: post
title: "Google Maps Component for iOS の CoordinateBounds について"
date: 2014-01-16 18:05
comments: true
categories: [Xamarin, iOS, C#, GoogleMapsAPI]
---
すごくニッチな話ですけど、Xamarin Components にある Google Maps を Xamarin.iOS で使った。
<!--more-->
* [Google Maps /Components](http://components.xamarin.com/view/googlemapsios)

このライブラリの中に ``CoordinateBounds`` という「領域」を表すクラスがある。これは [GMSCoordinateBounds](https://developers.google.com/maps/documentation/ios/reference/interface_g_m_s_coordinate_bounds?hl=ja) の Binding だ。

で、CoordinateBounds には ``Including`` ([includingCoordinate](https://developers.google.com/maps/documentation/ios/reference/interface_g_m_s_coordinate_bounds?hl=ja#a63cffdf310ca19c6bab74c9a4034aadd)) というメソッドがあって、これを呼ぶと指定した座標が入るように領域を拡幅してくれる、便利だ。

で、最初はこのクラスをこう使っていた。

```csharp
var bounds = new CoordinateBounds();
bounds.Including(new CLLocationCoordinate2D(34d, 134d));
bounds.Including(new CLLocationCoordinate2D(33d, 133d));
bounds.Including(new CLLocationCoordinate2D(35d, 135d));
```

動かしてみて、この使い方だと ``bounds`` から期待した結果が得られないことに気づいた。範囲が -180〜+180 になってしまった。

正しくはこう。

```csharp 
var bounds = new CoordinateBounds(
    new CLLocationCoordinate2D(34d, 134d),
    new CLLocationCoordinate2D(33d, 133d));
bounds.Including(new CLLocationCoordinate2D(35d, 135d));
```

これだと結果は、正しく [33,133 - 35,135] を返す。

処理上、生成時に２つの座標が揃ってないケースだったので、「あ、デフォルトコンストラクタあるじゃん」と使ってたらハマった。本家 iOS 版の方には引数無しの initXXX は無かった。

Objective-C の仕様上 alloc して init しないのを防げない、んだっけ？
だから、Xamarin.iOS の Binding でデフォルトコンストラクタを隠せないのかな？
突っ込んで調べてないけど、Binding ライブラリを使う時は注意しましょう、ちゃんと本家のAPIリファレンスを見ましょう、というお話でした。