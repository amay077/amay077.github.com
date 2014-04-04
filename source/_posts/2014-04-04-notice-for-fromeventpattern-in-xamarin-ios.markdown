---
layout: post
title: "Xamarin.iOS で FromEventPattern を使うときの注意点"
date: 2014-04-03 15:29
comments: true
categories: [Xamarin, iOS, C#, ReactiveExtensions]
---
Reactive Extensions の ``FromEventPattern`` を使うと、イベントを ``IObservable`` に変換できて、複数のイベントに時系列な関係を与えたり、他のストリーム処理とシームレスに扱えたりします。
<!--more-->

* [イベント・プログラミングとRx － ＠IT](http://www.atmarkit.co.jp/fdotnet/introrx/introrx_02/introrx_02_02.html)

Xamarin .iOS でも .Android でもこの機能を使うことができて大変便利ですが、Xamarin.iOS の場合 AOT による制限に気をつける必要があります。

以下は、なんの変哲もない、「ボタンを押したらタイトルを ”Clicked!” に変える」コードです。

```csharp
MyButton.TouchUpInside += (s, e) => MyButton.SetTitle("Clicked!", UIControlState.Normal);
```

これを FromEventPattern を使うとこう書けます。

```csharp DoesNotWorkOnDevice.cs
Observable.FromEventPattern(MyButton, "TouchUpInside")
.Subscribe(x => MyButton.SetTitle("Clicked!", UIControlState.Normal));
```

このコード、iOSシミュレータでは正常に動作しますが、 **実機では、ビルドは通りますが動作しません。** 実行時にこんなエラーがでます。


> System.InvalidOperationException: Could not find event 'TouchUpInside' on object of type 'MonoTouch.UIKit.UIButton'.

``TouchUpInside`` が無いと言われます。

これは AOT により生成されたコードに、このイベントが含まれないのだと推測します。イベント名を文字列リテラルで指定しているので、そこまでの解析は期待できないですよね。

シミュレータで動作したのは、この場合は AOT でなく JIT で動作しているため。以下でも言及されています。

* [Xamarin.iOSの仕組みとアプリケーションの構成 - Build Insider](http://www.buildinsider.net/mobile/insidexamarin/05)

> 対象がiOSシミュレーターである場合と、iOSデバイスである場合とで、大きく異なる。iOSシミュレーターは、エミュレーターではなく、あくまでMac OS Xが動作しているx86 CPUの上で動作している仮想マシンであり、アプリケーションはJITによって動作する。iOSデバイスはARMであり、iOSデバイス用にビルドされたアプリケーションはAOTによってARMのCPU命令に変換されており、ARM上でしか動作しない。

Xamarin.iOS では実機で動作させないと安心ならないと言われる所以です。

さて、このケースでは、FromEventPattern の別なオーバーロードを使うことで解決です。

```csharp WorkOnDevice.cs
Observable.FromEventPattern(
  h => MyButton.TouchUpInside+=h, 
  h => MyButton.TouchUpInside-=h)
.Subscribe(x => MyButton.SetTitle("Clicked!", UIControlState.Normal));
```

Xamarin.iOS の制限事項は以下に。

* [Limitations | Xamarin](http://docs.xamarin.com/guides/ios/advanced_topics/limitations/)

これまでこの制限に引っかかった事がなかったのですが、初めて引っかかりました。

メソッドを文字列リテラルで書いた時点で私の負けです、本当にありがとうございました。