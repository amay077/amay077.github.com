---
layout: post
title: "Google Maps SDK for iOS を Xamarin.iOS で使う"
date: 2013-04-22 22:13
comments: true
categories: [Xamarin, C#, iOS, GoogleMapsAPI]
---
Googleマップの iOS版とともに公開された [iOS用のMaps SDK](https://developers.google.com/maps/documentation/ios/) ですが、Xamarin.iOS でも使うことができます。(ちなみに Android用の Google Maps API v2 を Xamarin.Android で使う方法は[以前](http://amay077.github.io/blog/2013/03/05/xamarin-android-using-google-maps-android-api-v2/)書きました。)
<!--more-->
[Xamarin Component Store](http://components.xamarin.com/) にて Free で(ラッパー)ライブラリが公開されているので、それを使います。

コンポーネントをダウンロードすると、中にサンプルプロジェクトが含まれているのでそれを動かしてみます。

## 適当な Xamarin.iOS プロジェクトを作る

メニュー -> ファイル -> 新規 -> ソリューション -> C# -> iOS -> iPhone -> Single View Application を選択。ソリューションはなんでもいいです。どうせ後で捨てるので。テキトーに ``HogeApp`` とでもしましょうか。

## プロジェクトに Google Maps コンポーネントを追加する

メニュー -> プロジェクト -> Get More Components を選択します。

**Google Maps** を探しだして [Add to App] します。

![image1](https://dl.dropboxusercontent.com/u/264530/qiita/gmap_on_xamarin_ios_1.png)

しばらくすると、左側のビューの Components の中に "Google Maps" が作成されるので、それをダブルクリックして開きます。

右に表示されるコンテンツのタブから「Samples」を選択します。

![image2](https://dl.dropboxusercontent.com/u/264530/qiita/gmap_on_xamarin_ios_2.png)

さらに "iOS Sample" の「Open Sample」を押します。
すると、左側のビューに「GoogleMapsSample」プロジェクトが追加されます。

![image3](https://dl.dropboxusercontent.com/u/264530/qiita/gmap_on_xamarin_ios_3.png)

この時点で HogeApp は意味がなくなってしまいましたが、ディスクから削除すると GoogleMapsSample も一緒に消えてしまうので、無視しておきます。GoogleMapsSample がどこにあるかは、上図のように「親フォルダを開く」すると Finder で確認できます。

以降はこの GoogleMapsSample プロジェクトを使います。

## APIキーを取得する

ここからは、Component -> GoogleMaps をダブルクリックすると表示される「Getting Started」の「The Google Maps API Key」に沿っていきます。

#### 1. Create an API project in the [Google API Console](https://code.google.com/apis/console)

ブラウザで Google API Console へ移動して、左上のドロップダウンから「Create…」にてプロジェクトを作成します。既に存在しているなら、それを使ってもいいです。

#### 2. Select the Services pane in your API project, and enable the Google Maps SDK for iOS. This displays the Google Maps Terms of Service. 

左側のメニュー「Services」の中にある「Google Maps SDK for iOS」を ON にします。利用規約にも同意しましょう。

#### 3. Select the API Access pane in the console, and click Create new iOS key. 

左側のメニュー「API Access」を開き、ページの一番下に表示される「Create new iOS key」ボタンを押します。

#### 4. Enter one or more bundle identifiers as listed in your application's .plist file, such as com.example.myapp. 

入力欄に「com.example.myapp」とタイプします。本来はちゃんとした「あなたのアプリの固有ID」を入れるのでしょうが、当方 iOS 開発は詳しくないのでここでは例のままにします。

#### 5. Click Create. 

「Create」を押します。

#### 6. In the API Access page, locate the section Key for iOS apps (with bundle identifiers) and note or copy the 40-character *API key. 

すると、**Key for iOS apps** に 作成されたAPIキーが表示されるはずです。

![image4](https://dl.dropboxusercontent.com/u/264530/qiita/gmap_on_xamarin_ios_4.png)

この API key をコピーしておいて、Xamarin Studio に戻ります。

## Info.plist を開いて、Identifier を設定します

Google API Console で設定した Identifier と一致させる必要があるので「com.example.myapp」を設定します。(普通は先にアプリの Identifier を決めてから API key を取得するのでしょうが)

![image5](https://dl.dropboxusercontent.com/u/264530/qiita/gmap_on_xamarin_ios_5.png)

## AppDelegate.cs を開いて、API key を設定する

コピーしておいた API key を、下図の位置に貼り付けます。

![image6](https://dl.dropboxusercontent.com/u/264530/qiita/gmap_on_xamarin_ios_6.png)


## 動かす！

以上で準備完了です。
実行してみましょう。Android版の Google Maps API v2 は、エミュレータでは実行できませんが、iOS版は、シミュレータでも実行できます。

ばばーん！

![image7](https://dl.dropboxusercontent.com/u/264530/qiita/gmap_on_xamarin_ios_7.png)

あれ？ズームコントロールとか表示されないのね。

とりあえず動く所までできたので、今日はこの辺で。

## まとめ
* Google Maps SDK for iOS も Xamarin で普通に使える！
* Xamarin さん、Component に含まれるサンプルの「よりよい取り出し方」教えてください
* ``CLLocationCoordinate2D`` とか長ェよ、ただの ``LatLng`` だろ
* というように Google Maps Android API v2 とは設計思想は同じですが、インターフェースが違う(CLLocationCoordinate2D は CoreLocation のクラスですね)ので共通にはなりません。SDK が提供する機能にも差異があります。(さすがに Android > iOS です)