---
layout: post
title: "MvvmCross だけじゃない！クロスプラットフォームMVVMフレームワーク「QuickCross」を試す"
date: 2014-02-11 19:47
comments: true
categories: [Xamarin, Android, iOS, C#]
---
[以前、MvvmCross を紹介しました](http://amay077.github.io/blog/2013/12/25/using-mvvmcross-the-x-platform-mvvm-framework/) が、Xamarin で使える同種のフレームワークはまだいくつかあります。
<!--more-->
今回は、その中の一つ、「QuickCross」を見てみます

* [MacawNL/QuickCross](https://github.com/MacawNL/QuickCross)

## なぜ他のMVVMフレームワークが必要なのか？

[Why another cross-platform Mvvm framework?](https://github.com/MacawNL/QuickCross#why-another-cross-platform-mvvm-framework) にて、MvvmCross があるのになぜ？という事を説明しています。

要約すると、MvvmCross は、高機能だが Fat で複雑で、拡張が大変であるのに対し、QuickCross は、軽量で生産性が高く、拡張が簡単である、との事です。

## 主な機能

[Features](https://github.com/MacawNL/QuickCross#features) より。

* Xamarin.iOS, Xamarin.Android, Windows Phone, Windows Store Apps に対応。
* バイナリは使ってない！Snippet と、プロジェクトにソースコードの追加を行うだけです。
* ViewModel や View の追加は package manager console からコマンドを実行して行います。
* いくつかのコードスニペットを提供します。
* 以下略…

## 仕組み

![](https://raw.github.com/MacawNL/QuickCross/master/assets/quickcross_pattern.png) 
via https://github.com/MacawNL/QuickCross#features

Navigator って概念があるのが MvvmCross と違うとこですかね。

## 使い方

[Getting Started](https://github.com/MacawNL/QuickCross#getting-started) を見てください。

Nuget の Package manager console を使う必要があるので、Mac と Xamarin Studio 、そして Indie Edition では試せません、残念。
Visual Studio ＋ Xamarin Business Edition以上を使ってる方、試してみてください。

## サンプルを動かしてみた

Getting Started は試せませんでしたが、github に含まれるサンプルは Mac + Xamarin Studio でも動かせました。

[QuickCross.ios.sln](https://github.com/MacawNL/QuickCross/blob/master/QuickCross.ios.sln) を Xamarin Studio で開いて実行したところ↓

![](https://dl.dropboxusercontent.com/u/264530/qiita/introduce_quickcross_01.png)

MvvmCross と同じく、ViewModel などは Shared プロジェクトの方にあります。
Shared プロジェクトは PCL にできるんじゃないかなーと思いやってみましたが、

* Profile147(.NET4.0) では ``System.Windows.Input.ICommand`` が無いと言われ
* Profile78(.NET4.5) では、この[バグ](https://bugzilla.xamarin.com/show_bug.cgi?id=17247) にエンカウント

してビルドできませんでした、残念。

## まとめ

MvvmCross は確かに大規模すぎて使うのが大変です。拡張するには Plugin を自作する事になりますし。
QuickCross は、すべてのソースコードがプロジェクトにあるので、カスタマイズが手軽に行えそうだというのは分かりました。

Xamarin Starter Edition の場合、64kbyte までのバイナリ制限があるので、MvvmCross は使えませんが、QuickCross なら使えるかも知れません。

ただ残念なのは、Nuget の Package Manager Console を使う必要があるために、Visual Studio が必要で、その為には Xamarin も Business Edition 以上が必要になってしまう所です。

Xamarin Studio のみでも使えるくらい Lightweight だったら、もっと試してみたくなるフレームワークです。