---
layout: post
title: "Xamarin Component Store の紹介]"
date: 2013-12-10 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#]
---
Titanium に [MarketPlace](https://marketplace.appcelerator.com/home) があるように、Xamarin のマーケットプレイスとして Xamarin Components があります。
<!--more-->
* [Xamarin Components](http://components.xamarin.com/)

これは、Xamarin Studio とも連携していて、開発中にIDEから、コンポーネントを探す→ダウンロード＆組み込み、が速やかに行えるようになっています。

今日は、Components の中から主なもの、私が使ったことがあるものを少し紹介します。

### [Json.NET](http://components.xamarin.com/view/json.net)

.NET で Json を扱うのに必須のライブラリ。Nuget にもあるし、こっちにもあります。

### [ZXing.Net.Mobile](http://components.xamarin.com/view/zxing.net.mobile)

定番の1次元/2次元バーコードリーダーライブラリ。

### [SQLite.NET](http://components.xamarin.com/view/sqlite-net)

こちらも定番の SQLite 用ライブラリ。.NET ですからデータ・プロバイダ準拠の API です。

### Google Maps

Google Map を使うためのライブラリ。決して Google Maps JavaScript API のラッパじゃないですよ。

* iOS版 - [Google Maps](http://components.xamarin.com/view/googlemapsios)
* Android版 - [Google Play Services](http://components.xamarin.com/view/googleplayservices)

これらは過去に使い方の記事を書きました。

* [Xamarin.Android での Google Map(というか Play Service) 利用が、本家より簡単になった件](http://qiita.com/amay077/items/14191c808e9cac4eae2c)
* [Google Maps SDK for iOS を Xamarin.iOS で使う](http://qiita.com/amay077/items/db2c2d5d0060ba65a0e8)

### [Xamarin.Mobile](http://components.xamarin.com/view/xamarin.mobile)

アドレス帳、位置情報、カメラへの、プラットフォームに依存しない API を提供します。

### [Azure Mobile Service](http://components.xamarin.com/view/azure-mobile-services)

Azure を冠しているせいで Parse などに比べてイマイチ知名度がない(と勝手に思っている) Microsoft の BaaS を使うためのライブラリ。

### [Reactive Extensions (Rx) for Xamarin](http://components.xamarin.com/view/rxforxamarin)

LINQ を更に使い倒したいなら必須ですね。

### [Android iBeacon Service](http://components.xamarin.com/view/xamarin-android-ibeacon-service)

IBeacon を Xamarin.Android で使えるようにするライブラリ。[android-ibeacon-service](https://github.com/RadiusNetworks/android-ibeacon-service) の Javaバインディング。

## Components と PCL

Components を使う上での注意点です。

最近、PCL が Xamarin.iOS と Android に対応しましたが、この Components で配布されているライブラリは、PCL とは限りません。

Json.NET, ZXing, SQLite とか、PF固有ロジックを含まないので同一バイナリでいけそうなものですが、実際にそうなっているかは分かりません（たぶん参照してるアセンブリをアセンブリブラウザで確認すればわかると思うけど）。ので、自分のアプリでPF依存を減らしたいならば、Nuget から PCL版を持ってくるか、ソースを入手して自分でビルドする必要があります。

Xamarin.Mobile も、各PFで使い方は全く同じながら、共通なデータクラスが各PF毎のDLLにパッケージされてしまっているので、同じバイナリでは使えません。これの分離については後日書きます。

Rx は…自前でビルドしようと挑戦しましたが、``IObservable`` は誰のもの？で躓いて早々に諦めました（^_^;

## More…

企業やコミュニティが公開している .NET のライブラリも Xamarin で使えるかも知れません。画面が絡むものは確実にムリですが、通信や計算に特化したものはそのまま、あるいは軽微な修正のみで使える可能性があります。

Xamarin では、既存の DLL や exe がどのくらい再利用できるかを計測する “Mobility Scanner” を公開しています。

* [How mobile is your .NET?] (http://scan.xamarin.com/)
* [Blog post](http://blog.xamarin.com/how-mobile-is-your-.net-code/)

高得点が出るライブラリなら、そのまま使えるんじゃないでしょうか。

## まとめ

とここまで書いて、過去にこんな

* [Xamarin Component Store を眺めてみる](http://qiita.com/amay077/items/811cdd8ab3d1243045b6)

記事を書いたのを思い出しました。内容ダブってますがまあいいや、リファインってことで。
