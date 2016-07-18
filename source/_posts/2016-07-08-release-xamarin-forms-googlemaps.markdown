---
layout: post
title: "Xamarin.Forms 向けの地図ライブラリ「Xamarin.Forms.GoogleMaps」をリリースしました"
date: 2016-06-27 23:59:00 +0900
comments: true
categories: [Xamarin, Xamarin.Forms, Android, iOS, GoogleMapsAPI, ReleaseNotes, Xamarin.Forms.GoogleMaps]
---
Xamarin.Forms で使える地図ライブラリは、公式が出している

* [Xamarin.Forms.Maps](https://www.nuget.org/packages/Xamarin.Forms.Maps/) - Maps models and renderers for Xamarin.Forms

があるのですが、非常に機能が少ないです（ピンがおけるだけで、図形の描画すらできません）。
<!--more-->

なので、別な選択肢としての「Xamarin.Forms向け地図ライブラリ」を開発し始めました。

それがこちら

* [Xamarin.Forms.GoogleMaps](https://www.nuget.org/packages/Xamarin.Forms.GoogleMaps/) - Yet another Maps library for Xamarin.Forms that optimized for Google maps.

です。

iOS では MapKit に代わり [Google Maps SDK for iOS](https://developers.google.com/maps/documentation/ios-sdk/?hl=ja) を使用し、 Android/iOS 共に Google Maps に特化することで、API の共通化を容易にし、恐らくAPI共通化の足枷になっているであろう UWP(Bing maps) のサポートは最小限に留めています。

![screenshot1](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_forms_googlemaps_intro_01.png)


## 現在の機能

現在のバージョンは 1.1.0 。
公式の Xamarin.Forms.Maps に比べて、ライン・ポリゴン・円を追加できるようになりました。

![screenshot2](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_forms_googlemaps_intro_02.png)

詳しい比較は

* [Xamarin.Forms.Maps との比較 - Xamarin.Forms.GoogleMaps Wiki](https://github.com/amay077/Xamarin.Forms.GoogleMaps/wiki/Xamarin.Forms.Maps-%E3%81%A8%E3%81%AE%E6%AF%94%E8%BC%83)

にあります。

## サンプルプログラム

* [Xamarin.Forms.GoogleMaps/XFGoogleMapSample - github](https://github.com/amay077/Xamarin.Forms.GoogleMaps/tree/master/XFGoogleMapSample)

にあります。

Google Maps の APIキーを Android / iOS それぞれで取得する必要があります。

* Android -  [Xamarin.Androidで地図を表示するには？（Google Maps使用） - Build Insider](http://www.buildinsider.net/mobile/xamarintips/0020)
* iOS - [Google Maps SDK for iOS  |  Google Developers](https://developers.google.com/maps/documentation/ios-sdk/?hl=ja) の「クイック スタート ステップ」
をそれぞれ参照してください。

## オープンソース

[Xamarin Open Source SDK](http://open.xamarin.com/) により、 Xamarin.Forms のソースコードもオープンソースになったので、 [Xamarin.Forms/Xamarin.Forms.Maps - github](https://github.com/xamarin/Xamarin.Forms/tree/master/Xamarin.Forms.Maps) などを Fork して作りました。

このライブラリ自体もオープンソースであり、

* [Xamarin.Forms.GoogleMaps: Map library for Xamarin.Forms using Google maps API](https://github.com/amay077/Xamarin.Forms.GoogleMaps)

で開発しています。（スターを付けてもらえると作者がよろこびます）

[要望、コメントなど](https://github.com/amay077/Xamarin.Forms.GoogleMaps/issues)もらえると嬉しいです。よろしくおねがいします。
