---
layout: post
title: "Xamarin.iOS で Yahoo! iOSマップSDK を使ってみる"
date: 2013-12-23 0:00
comments: true
categories: [Xamarin, XAC13, iOS, C#, Geo]
---
Xamarin.iOS では、既存の iOS用ライブラリが利用できます。
今日は例として Yahoo Maps SDK for iOS を Xamarin.iOS から使ってみます。
<!--more-->
## Yahoo! iOSマップSDK とは

* [YOLP(地図):Yahoo! iOSマップSDK - Yahoo!デベロッパーネットワーク](http://developer.yahoo.co.jp/webapi/map/openlocalplatform/v1/iphonesdk/)

Yahoo! Japan が提供する地図SDKです。Google Maps にはない魅力として、「雨雲レーダー」「経路探索/案内」「AR機能」が挙げられます。
利用するには上記サイトから APIキー の発行が必要です。

## Xamarin.iOS で Objective-C ライブラリを使う方法

* [Binding Objective-C | Xamarin](http://docs.xamarin.com/guides/ios/advanced_topics/binding_objective-c/)

「Binding」と呼ばれます。
Obj-Cライブラリを呼び出すラッパーのようなものを C# で記述し、それを Xamarin.iOS アプリケーションから使用することが出来ます。

## Xamarin.iOS で Yahoo! iOSマップSDK を使う手順

### 1. Yahoo! iOSマップSDK をダウンロードし、APIキー を発行する

SDK のダウンロードは、

* [YOLP(地図):Yahoo! iOSマップSDK - Yahoo!デベロッパーネットワーク](http://developer.yahoo.co.jp/webapi/map/openlocalplatform/v1/iphonesdk/)

です。APIキー の発行は、

* [アプリケーションの管理：Yahoo!デベロッパーネットワーク](https://e.developer.yahoo.co.jp/dashboard/)

です。手順は、

* [YOLP(地図):利用準備 - Yahoo!デベロッパーネットワーク](http://developer.yahoo.co.jp/webapi/map/openlocalplatform/v1/iphonesdk/tutorial1.html)

が分かりやすいです。

### 2. Xamarin.iOS で Binding プロジェクトを作る

Xamarin Studio で新しいソリューションを作ります。
iOS → iOS Binding Project で、名前は 「YMapBinding」、ソリューション名は「YMapSample」とします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_ymapsdk_on_xamarin_ios_01.png)

### 3. Yahoo! iOSマップSDK のライブラリファイルを Binding プロジェクトに入れる

ダウンロードした Yahoo! iOSマップSDK を解凍して、中に含まれる ``YMapKit`` ファイルを ``libYMapKit.a`` にリネームします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_ymapsdk_on_xamarin_ios_02.png)

Xamarin Studio で、YMapBinding プロジェクトに ``libYMapKit.a`` を追加します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_ymapsdk_on_xamarin_ios_03.png)

### 4. Yahoo! iOSマップSDK の API定義を C# で書く

Binding プロジェクトにある ``ApiDefinition.cs`` を以下のように書き換えます。

```csharp ApiDefinition.cs
using System;
using System.Drawing;
using MonoTouch.ObjCRuntime;
using MonoTouch.Foundation;
using MonoTouch.UIKit;

namespace YMapBinding
{
    [BaseType (typeof (UIView))]
    public partial interface YMKMapView {
        [Export ("initWithFrame:appid:")]
        IntPtr Constructor (RectangleF frame, string appid);
    }
}
```

よく目を凝らすと分かるんですが、これは iOS版ライブラリの ``initWithFrame:appId:`` というコンストラクタを C# で定義しています。
他のメソッドやプロパティ、イベント(delegate)も同じように定義するのですが、ここでは省略します。

次に ``libYMapKit.linkwith.cs`` を開いて、以下のように書き換えます。

```csharp libYMapKit.linkwith.cs
using System;
using MonoTouch.ObjCRuntime;

[assembly: LinkWith ("libYMapKit.a", LinkTarget.ArmV7 | LinkTarget.ArmV7s | LinkTarget.Simulator ,ForceLoad = true, 
    Frameworks="UIKit SystemConfiguration CoreGraphics CoreLocation Foundation OpenGLES QuartzCore")]
```

Yahoo! iOSマップSDK が依存するライブラリを Frameworks に列挙しています。

これでひとまず Binding プロジェクト 側は終わりです。

#### 5. サンプルアプリケーションプロジェクトを作る

YMapSample ソリューションに 新しいプロジェクト を追加、iPhone の Single view Application 、名称は「YMapApp」とします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_ymapsdk_on_xamarin_ios_04.png)

YMapApp プロジェクトを右クリックして、スタートアッププロジェクトをこちらに変更しておきます。

次に、参照設定で YMapBinding を追加します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_ymapsdk_on_xamarin_ios_05.png)

#### 6. Yahoo! iOSマップSDK を表示するコードを書く

YMapApp のビューコントローラ(たぶん YMapAppViewController)の``ViewDidLoad`` に、以下のように追記します。

```csharp YMapAppViewController.cs
public override void ViewDidLoad()
{
    base.ViewDidLoad();
    
    var v = new YMKMapView(new RectangleF(0f, 0f, 320f, 320f), 
        “<your app key>”);  // あなたが取得した APIキー
    this.View.AddSubview(v);
}
```

ああ、 ``using YMapBinding;`` も必要ですね。

アプリの方もとりあえずこれで OK。

#### 7. 動かしてみる

YMapApp を iOSシミュレータで実行してみます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_ymapsdk_on_xamarin_ios_06.gif)

はい、このように「とりあえず」Yahoo! iOSマップSDK を Xamarin.iOS で動かすことができました。

## この後（やること多いよ）

### API定義をちまちまと移植

この例ではコンストラクタ1つしか定義しませんでしたが、これを他のコンストラクタ、メソッド、プロパティ、イベントについて行う必要があります。これを助けるツールとして [Objective Sharpie](http://docs.xamarin.com/guides/ios/advanced_topics/binding_objective-c/objective_sharpie/) が公開されていますが、あまり期待しない方が良さそうです。Yahoo! iOSマップSDK をこのツールにかけてみましたが、出来上がった定義ファイルはエラーがたくさん出ました。

* [Binding Objective-C | Xamarin](http://docs.xamarin.com/guides/ios/advanced_topics/binding_objective-c/)

を理解した上で、Objective Sharpie の結果を参考にして、作っていく必要がありそうです。

### ApiDefinition.cs への定義は正しいのか？

「Frameworks に、Yahoo! iOSマップSDK が依存しているライブラリを列挙」してみましたが、[チュートリアル](http://developer.yahoo.co.jp/webapi/map/openlocalplatform/v1/iphonesdk/tutorial1.html) には ``libxml2.2.dylib`` も含まれています。が、 ``ApiDefinition.cs`` にはこれは記述していません。けど動いています。何かの機能を使った時に問題になるかもしれません。そしてこの .dylib という拡張子の場合にどう定義すれば良いのか不明です。

### ライブラリが使用するリソースはどこに？

ダウンロードした Yahoo! iOSマップSDK には ``image`` ディレクトリがあり、これをアプリケーションプロジェクトに配置することで、ライブラリがリソースを使うことになっています。Xamarin.iOS Binding プロジェクトではこれはどこに配置すればよいか、未調査です。

ということで Xamarin.iOS の Binding について紹介しました。
.Android の Binding は jar を放り込めばある程度自動で定義を生成してくれていたのに対し、かなり面倒な感じです。アプリケーションに必要な機能だけを定義して使っていく感じかなあと感じました。