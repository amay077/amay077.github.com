---
layout: post
title: "Xamarin.iOS でインターネット通信が可能かを取得する"
date: 2016-08-26  23:59:59 +0900
comments: true
categories: [Xamarin, Xamarin.iOS, iOS, C#]
---

Xamarin.iOS で、「端末からインターネット通信が可能か？」を調べたい。

<!--more-->

Objective-C だと [Reachability](https://github.com/tonymillion/Reachability)、swift だと [Reachability.swift](https://github.com/ashleymills/Reachability.swift) を使うようだけど、 Xamarin.iOS ではどうするか？

* [Swift でネットワーク状況を調べる - メモ用紙](http://d.hatena.ne.jp/scientre/20150527/get_network_status_in_swift)

Bindingライブラリがあるのかな？と思ったら、サンプルで C# のソースコードが提供されていた。

* [Detect if Network is Available - Xamarin](https://developer.xamarin.com/recipes/ios/network/reachability/detect_if_network_is_available/)

の [``reachability.cs``](https://github.com/xamarin/monotouch-samples/blob/master/ReachabilitySample/reachability.cs) がそれ。
（名前から察して Reachability.swift を C# で書きなおしたもの？詳しくは見てないけど。）

で、このサンプルの ``Reachability`` を使うと、以下のような感じで、「インターネット通信が可能か？」を調べられる。

```csharp
public bool IsNetworkAvailable
{
    get
    {
        return Reachability.InternetConnectionStatus() != NetworkStatus.NotReachable;
    }
}
```

WiFi とかセルラーとか細かいステータスもあるので、あとはコードを見てください。
