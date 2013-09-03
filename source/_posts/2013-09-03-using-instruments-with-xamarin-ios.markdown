---
layout: post
title: "Xamarin.iOS + iOS Simulator で Instruments を使う"
date: 2013-09-03 21:29
comments: true
categories: [Xamarin, iOS]
---
Xamarin.iOS(旧MonoTouch)では、Xcode のプロファイラである Instruments がそのまま使えます。
<!--more-->
## デモプロジェクト

* [Instruments Walkthrough | xamarin](http://docs.xamarin.com/guides/ios/deployment%2C_testing%2C_and_metrics/instruments_walkthrough)

にある ``MemoryDemo.zip`` をダウンロード、解凍します。2つプロジェクトが含まれていますが、 ``before`` を使います。

## 手順

### 1. プロジェクトをとりあえず実行

Xamarin Studio で ``before/MemoryDemo.sln`` を開いて、とりあえずビルド、Simulator で実行します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_instruments_with_xamarin_ios_01.png)

上下にスクロールすると、次々と画像を読み込むので Allocate がハンパないよ、ってデモのようです。

### 2. Xamarin Studio から Instruments を起動する。

メニュー - ツール - Launch Instruments で起動します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_instruments_with_xamarin_ios_02.png)

起動はしましたが、勝手にアプリが実行されるわけではありません。ここから少しだけ面倒な手続きが必要です。

### 3. iOS Simulator にインストールしたアプリを指定して Instruments を実行する

Instruments を起動すると、下のような画面になっています。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_instruments_with_xamarin_ios_03.png)

左メニューから [iOS Simulator] - [Memory]、右から [Allocations] を選び [Choose] ボタンを押します。

次に、[Target] をクリックして、[Choose Target] - [Choose Target…] と進みます。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_instruments_with_xamarin_ios_04.png)

下のような画面になります。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_instruments_with_xamarin_ios_05.png)

次に Finder を起動して、iOS Simulator のディレクトリへ移動します。

iOS Simulator のディレクトリは通常、``~/Library/Application Support/iPhone Simulator/`` です。さらにアプリ毎に GUID で分けられているので目的のアプリを探してください。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_instruments_with_xamarin_ios_06.png)

アプリのディレクトリを開いたら、その中のアプリケーションファイル(ここでは ``MemoryDemo``) を、先ほど開いておいた Instruments の中へドラッグ＆ドロップします。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_instruments_with_xamarin_ios_07.png)

そして [Choose] を押すと、Target が MemoryTest になっているのが分かります。

これでようやく実行できます。赤い●を押します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_instruments_with_xamarin_ios_08.png)

と、iOS Simulator で MemoryTest が実行され、Instruments でプロファイルしている事が確認できます。Simulator でグリグリスクロールすると、Allocations がガンガン増えてく様子が分かります。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_instruments_with_xamarin_ios_09.png)

### 3. 2度目以降は？

Instruments でもう一度赤い●を押すと停止します。アプリを更新する時は、Xamarin Studio 側でビルド-実行して iOS Simulator のアプリファイルを更新してから、Instruments で再度、赤い●を押します。

もし Instruments を終了してしまっても、最近使ったアプリは Choose Target に最近使ったアプリとして残るので、またドラッグ＆ドロップすることはありません。

## まとめと参考

Xamarin.iOS+iOS Simulator での Instruments の導入部分を説明しました。

下に紹介するサイトが公式の情報です。
ここには、実機にインストールしたアプリのプロファイル方法や、Instruments の使い方などが説明されているので合わせてどうぞ。

* [Profiling Xamarin.iOS Applications with Instruments | xamarin](http://docs.xamarin.com/guides/ios/deployment%2C_testing%2C_and_metrics/using_instruments_to_detect_native_leaks_using_markheap)
* [Instruments Walkthrough | xamarin](http://docs.xamarin.com/guides/ios/deployment%2C_testing%2C_and_metrics/instruments_walkthrough)
* [iOS Simulator Help: Setting Instruments to Launch an iOS App in Simulator](https://developer.apple.com/library/ios/recipes/instruments_help-launch-into-simulator-help/LaunchIntoSimulator.html)

※ [Instruments Walkthrough](http://docs.xamarin.com/guides/ios/deployment%2C_testing%2C_and_metrics/instruments_walkthrough) の No.14 の [この画像](http://docs.xamarin.com/static/guides/ios/deployment,_testing,_and_metrics/instruments_walkthrough/Images/05_related_code.png) には、Instruments に Xamarin.iOS(C#) のソースコードが表示されているように見えるんだけど、これどうやるのかなあ。。手順通り動かしたつもりが出てこない。。。SourceMap の設定みたいなのが要るのかなあ。 
