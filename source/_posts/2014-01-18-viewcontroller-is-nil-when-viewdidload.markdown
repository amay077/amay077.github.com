---
layout: post
title: "ViewController の ViewDidLoad で this が nil になった件"
date: 2014-01-18 18:18
comments: true
categories: [Xamarin, iOS, C#]
---
Xamarin Studio + Xcode で Storyboard でアプリを作っていた。複数ある画面の内、ある一つの画面に遷移すると落ちる現象に見舞われていろいろ調べていた。
<!--more-->
Xamarin Studio で該当画面の ViewController の ViewDidLoad にブレークポイントを仕掛けて停止させ、ウォッチしてみたところ、なんと this が「nil」になっていた。

これのおかげで、ViewController に配置した UILabel などにもアクセスできない。

ViewController を作りなおしてみたり、呼び出し方法を変えてみたりいろいろやってみたけど解消せず。

30分ほど悪戦苦闘した後、実機にインストールされている該当アプリを一旦削除し、Xamarin Studio も終了させた後に再起動、ソリューションをクリーンして再ビルドして実行してみたところ、問題が解消した。

なにがしかのトラブルが起きた時は、まずは端末内のアプリを消してみると良いのかも知れない。（これまでの経験的に）