---
layout: post
title: "ReactiveX と MVVM と DataBinding の関係について図にしてみた"
date: 2014-10-30 01:17:39 +0900
comments: true
categories: [mvvm, reactiveextensions, rxjava, reactivecocoa, xamarin, android, ios, c#]
---
* [ReactiveCocoa Tokyo #rac_tokyo - connpass](http://connpass.com/event/8680/)
* [RxJava Night #rxjnight - connpass](http://connpass.com/event/9061/)
* [#10 node.js sideshow | mozaic.fm](http://mozaic.fm/post/100741841543/10-node-js-sideshow)

<!--more-->

などなどをいろいろ眺めておりまして（東京うらやましい）、Reactive Extensions とか MVVM とかいろいろ熱い！楽しい！と思っているわけですが、いろいろなライブラリがあって、それらのカバーする範囲がいまいち明確になってない気がしたので、自分なりに整理してみました。

![](https://dl.dropboxusercontent.com/u/264530/qiita/reactive_mvvm_databindings_relations_01.png)

MVVM っていうと、Messenger とか DIコンテナ的なものとかもあるわけですが、主に DataBinding と Rx の違いにフィーチャーしたかったので除外しました。

　DataBinding は、[DependencyProperty](http://www.atmarkit.co.jp/ait/articles/1008/03/news097_3.html) や [BindableProperty](http://blog.falafel.com/learning-xamarin-custom-renderers-in-xamarin-forms/) みたいなものがあるかどうかという感じで考えていて、「XAMLとかのマークアップでバインディング指定できなければならない」という考えではないです。

　View, DataBinding, ViewModel, ReactiveX の各ブロックは基本的にはどの組み合わせでもよくて（特に ReactiveX は他とは別の世界のものなので）、しかし中には ReactiveProperty のように ReactiveX に依存しつつ ViewModel の機能を提供するものがあったり、ReactiveCocoa のように「全部入り」のものがあったりします。また、View と ViewModel を繋ぐためにはなんらかの DataBinding が必要です。

という理解なんですが、あってますかね？

　私は Xamarin 推しの人なので、 **Xamarin.Forms + ReactiveProperty が、MVVM+Rx のパワーをフル活用できて、しかも iOS/Android で大部分のコードが共有できるという最強の組み合わせなんですよ！** というのを言いたいわけです。

## Links

* Xamarin.Forms - [Build a Native Android UI & iOS UI with Xamarin.Forms - Xamarin](http://xamarin.com/forms)
* Prism - [patterns & practices: Prism - Download: Prism 5.0 for .NET 4.5](http://compositewpf.codeplex.com/releases/view/117297)
* MVVM Light Toolkit - [MVVM Light Toolkit - Home](https://mvvmlight.codeplex.com/)
* Reactive Extensions - [Rx (Reactive Extensions) - Home](https://rx.codeplex.com/)
* ReactiveProperty - [ReactiveProperty - MVVM Extensions for Rx - Home](https://reactiveproperty.codeplex.com/)
* ReactiveUI - [reactiveui/ReactiveUI](https://github.com/reactiveui/reactiveui)
* MvvmCross - [MvvmCross/MvvmCross](https://github.com/MvvmCross/MvvmCross)
* RxJava - [ReactiveX/RxJava](https://github.com/ReactiveX/RxJava)
* RxAndroid - [ReactiveX/RxAndroid](https://github.com/ReactiveX/RxAndroid)
* android-binding - [gueei/AndroidBinding](https://github.com/gueei/AndroidBinding)
* ReactiveCocoa - [ReactiveCocoa/ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
