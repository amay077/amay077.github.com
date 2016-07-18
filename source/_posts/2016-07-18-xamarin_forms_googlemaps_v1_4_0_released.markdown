---
layout: post
title: "カスタムマーカーに対応した Xamarin.Forms.GoogleMaps v1.4.0 をリリースしました"
date: 2016-07-18  17:00:00 +0900
comments: true
categories: [Xamarin, Xamarin.Forms, Android, iOS, GoogleMapsAPI, ReleaseNotes, Xamarin.Forms.GoogleMaps]
---
Xamarin.Forms.GoogleMaps v1.4.0 をリリースしました。

<!--more-->

* [NuGet Gallery | Xamarin.Forms.GoogleMaps 1.4.0](https://www.nuget.org/packages/Xamarin.Forms.GoogleMaps/1.4.0)

待望？のカスタムマーカーに対応しました。

![image001](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_forms_googlemaps_v1_4_0_released_001.png)

## カスタムマーカーの使い方

``Pin.Icon`` に ``BitmapDescriptorFactory`` により生成される ``BitmapDescriptor`` を設定します。

``BitmapDescriptorFactory`` の３つのファクトリメソッドにより、
「デフォルトマーカーの色を変える」「プラットフォーム毎の画像リソースを使用する」「共通の画像リソースを使用する」
の３つが使用できます。

### デフォルトマーカーの色を変える

[``iOS:GMSMarker.markerImageWithColor``](https://developers.google.com/maps/documentation/ios-sdk/reference/interface_g_m_s_marker.html?hl=ja#ae320cb082a68c22eb1f37955f8e56228), [``Android:BitmapDescriptorFactory.defaultMarker``](https://developers.google.com/android/reference/com/google/android/gms/maps/model/BitmapDescriptorFactory.html?hl=ja#defaultMarker(float)) に対応する、既定のマーカー形状の色のみを変える機能です。

以下のように、 ``BitmapDescriptorFactory.DefaultMarker(Color)`` メソッドを使用します。

```csharp
pin.Icon = BitmapDescriptorFactory.DefaultMarker(Color.Pink);
```

### プラットフォーム毎の画像リソースを使用する

[``iOS:UIImage.imageNamed``](https://developers.google.com/maps/documentation/ios-sdk/marker?hl=ja#_7), [``Android:BitmapDescriptorFactory.fromAsset``](https://developers.google.com/android/reference/com/google/android/gms/maps/model/BitmapDescriptorFactory.html?hl=ja#defaultMarker(float)) に対応する、プラットフォーム固有の画像リソースを、マーカー画像として使用する機能です。

以下のように、 ``BitmapDescriptorFactory.FromBundle(string)`` メソッドを使用します。

```csharp
pin.Icon = BitmapDescriptorFactory.FromBundle("image01.png");
```

引数の bundleName は、 **同じ名称で** 、プラットフォーム毎に次のように用意されている必要があります。

#### Android の場合

Android側のプロジェクトの ``Assets`` ディレクトリ内に ``image01.png`` を追加し、 Build Action を ”Android Asset” に設定します。

![image001](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_forms_googlemaps_v1_4_0_released_002.png)

#### iOS の場合

iOS側のプロジェクト ``image01.png`` を追加し、 Build Action を ”BundleResource” に設定します。

![image001](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_forms_googlemaps_v1_4_0_released_003.png)

### 共通の画像リソースを使用する

画像を ``Stream`` を直接指定できる、機能です。

以下のように、 ``BitmapDescriptorFactory.DefaultMarker(Color)`` メソッドを使用します。

```csharp
// PCLプロジェクトに EmbeddedResouece として追加した "marker01.png" を読み込んで Stream 化
var assembly = typeof(CustomPinsPage).GetTypeInfo().Assembly;
var stream = assembly.GetManifestResourceStream($"XFGoogleMapSample.marker01.png");

// Stream をマーカーに設定
pin.Icon = BitmapDescriptorFactory.FromStream(stream);
```


## 【注意】プラットフォーム毎のマーカーサイズの違い

「プラットフォーム毎の画像リソースを使用する」「共通の画像リソースを使用する」で見られる現象なのですが、iOS と Android では **同じサイズの画像を指定しているのに iOS の方が大きく描画されます**

![image001](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_forms_googlemaps_v1_4_0_released_004.png)

なぜこうなるのか分かりませんが、 [Google Maps SDK for iOS](https://developers.google.com/maps/documentation/ios-sdk/intro?hl=ja) と [Google Maps Android API](https://developers.google.com/maps/documentation/android-api/intro?hl=ja) を直接使用した時もこうなるので、両者の仕様というかプラットフォーム自体の仕様なのかも知れません。

特に ``BitmapDescriptorFactory.FromStream`` を使用した時には使い勝手が悪いのですが、「ネイティブの Google Maps SDK の機能を共通APIでラップする」のが第一目標なので、とりあえず画像を何も加工しない実装でリリースしました。

何か原因などについてヒントがある方は

* [Why custom pin icon in iOS bigger than Android? · Issue #40 · amay077/Xamarin.Forms.GoogleMaps](https://github.com/amay077/Xamarin.Forms.GoogleMaps/issues/40)

に書いてもらえると助かります（日本語で大丈夫です）。

## サンプルプログラム

[XFGoogleMapSample](https://github.com/amay077/Xamarin.Forms.GoogleMaps/tree/master/XFGoogleMapSample) の [``CustomPinsPage.xaml.cs``](https://github.com/amay077/Xamarin.Forms.GoogleMaps/blob/master/XFGoogleMapSample/XFGoogleMapSample/CustomPinsPage.xaml.cs) でこれらの機能を使用しています。

是非使ってみてください。よければ [Github リポジトリ](https://github.com/amay077/Xamarin.Forms.GoogleMaps) に Star ください。
