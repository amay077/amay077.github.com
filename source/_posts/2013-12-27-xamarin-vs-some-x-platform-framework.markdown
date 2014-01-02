---
layout: post
title: "Xamarin vs 他のクロスプラットフォーム開発ツール"
date: 2013-12-05 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#]
---
Xamarin は、クロスプラットフォーム開発ツールとして紹介されることが多いので、他の同類のツールと比較してみたいと思います。

対象は Android と iOS に絞ります。
速度がーとか、メモリ使用量がー、とか言った話はナシで、仕組みとかを淡々と述べます。

ゲーム系は分からないのでナシで。業務アプリつくるレベルだと思ってください。

あ、いずれもガッツリ使ったことはないので、間違ってたらツッコミお願いします。

## 1. まず [Xamarin](http://xamarin.com/) についておさらい

### 開発言語

C#、F# などの .NET 言語を使います。VB.NET は…プロジェクトテンプレートが無いだけですかね？

### 実行モデル

Android では、パッケージ(apk) として Monoランタイムと.NET中間言語がパッケージングされ、アプリ実行時にネイティブコードにコンパイルされて実行されます、いわゆる JIT。

iOS では、 .NET言語は パッケージ(ipa) 作成時にネイティブコードにコンパイルされ、アプリ実行時にはネイティブコードが実行されます。事前コンパイル AOT(Ahead-Of-Time)コンパイルと呼ばれます。

UIパーツについては Android も iOS もネイティブのものが使われます。

### 共通にできる所とできない所

コアロジックは共通化できます。

画面、GPSやカメラなどプラットフォーム毎にAPIが異なる箇所は共通化できません。

### ネイティブ機能の呼び出し

※ここでいう「ネイティブ」とは、iOS なら Objective-C、Android なら Java で開発された部品、の意です。C言語のガチなやつじゃないです。

各PF製の外部ライブラリは Binding という仕組みで呼び出すことができます。


## 2. [Titanium Mobile](http://www.appcelerator.com/titanium/)

思えばクロスプラットフォーム開発ツールでもっとも先に普及しましたね。

### 開発言語

JavaScript を使います。

### 実行モデル

Android, iOS ともにパッケージには、Javascriptコードが含まれます。実行時にそれは、Javascriptエンジン(V8など)で解釈され、ネイティブAPIにブリッジされて実行されます。(あってる？

UIパーツについては Android も iOS もネイティブのものが使われます。

### 共通にできる所、できない所

コアロジック、および GPS やセンサーなど、Titanium によって共通APIが用意されている機能は共通化ができます。UI も Label や EditBox など、簡素なものは共通になります。

PF固有の UIパーツ は共通化できません。CoverFlowView は iOS でしか使えません。

### ネイティブ機能の呼び出し

各PF製の外部ライブラリは Module という仕組みで呼び出すことができます。


## 3. [Adobe AIR for モバイル](http://help.adobe.com/ja_JP/air/build/WSfffb011ac560372f-5d0f4f25128cc9cd0cb-8000.html)

正式名称なんですか？いわゆる Flash の AIR のモバイル版です。

### 開発言語

ActionScript を使います。

### 実行モデル

Android では、パッケージ(apk) として AIRランタイムとActionScript言語がパッケージングされ、アプリ実行時にネイティブコードにコンパイルされて実行されます。(AIRランタイムは別アプリとして切り離すこともできます)

iOS では、 ActionScript言語は パッケージ(ipa) 作成時にネイティブコードにコンパイルされ、アプリ実行時にはネイティブコードが実行されます。

UIパーツについては、Android/iOS のネイティブUIは使わず、AIR で用意されたパーツを使います。

### 共通にできる所、できない所

基本的に全てのロジック、UI が共通化できます。(機種依存や特定端末への最適化を除けば)

その替り AIR に用意されていない機能（例えば Android の Toast など）を使いたい場合、ネイティブ機能の呼び出しに頼ることになります。

### ネイティブ機能の呼び出し

各PF製の外部ライブラリは Native Extensions という仕組みで呼び出すことができます。


## 4. [Delphi XE](http://www.embarcadero.com/jp/products/delphi)

2010年あたりから Delphi "XE(X-platform Edition)" として、クロスプラットフォーム対応が可能になっています。(Android は最近対応しました)

### 開発言語

Delphi を使います。

### 実行モデル

Android/iOS ともにパッケージングの際にネイティブコードにコンパイルされます。

UIパーツについては、Android/iOS のネイティブUIは使わず、Delphi(というか FireMonkey）で用意されたパーツを使います。

### 共通にできる所、できない所

基本的に全てのロジック、UI が共通化できます。(機種依存や特定端末への最適化を除けば)

その替り Delphi(FireMonkey) に用意されていない機能（何があるのだろう？）を使いたい場合、ネイティブ機能の呼び出しに頼ることになります。

### ネイティブ機能の呼び出し

iOSapi.Foundation や、Androidapi.Log, Macapi.AppKit などの「Unit」を ``uses  して使うことができます。
([コメント](http://qiita.com/amay077/items/01917ef1be3da9259348#comment-eeefbb22b5318dede221)にて教えて頂きました)

## 5. [PhoneGap](http://phonegap.com/)

認知度高いので、一応。

### 開発環境

JavaScript を使います。

### 実行モデル

Android/iOS とも、アプリに見えて実際は WebView の上で JavaScript コードが動いてます。

### 共通にできる所、できない所

WebView 上で動くので基本共通なはずです。細かい所はよく分かってません。

### ネイティブ機能の呼び出し

Plugin というのを自作することで実現可能なようです。

## 6. [Qt(キュート) Mobile](http://qt.digia.com/Qtmobileedition)

C++ で書くんですよね？ってくらいしか分かりませんすいません。


## 参考

### Xamarin

* [How it Works - Xamarin](http://xamarin.com/how-it-works)
* [Xamarin 2.0 | ++C++; // 未確認飛行 C ブログ](http://ufcpp.wordpress.com/2013/02/24/xamarin-2-0/)
* [How Xamarin.Android works [ja]](https://docs.google.com/presentation/d/1QS9gWRGIzTiXqD9GK3X4JsskskHp2NcNBQJBly96Eis)

### Titanium Mobile

* [福井スマートフォンハッカソン Titanium Mobileの紹介](http://www.slideshare.net/MoriShingo/titanium-mobile-13081280)
* [Titanium Seminar » 種々のアプリ開発手法とTitaniumの優位性](http://titanium_seminar.soracid.com/219)

### Adobe AIR for Mobile

* [Adobe Flex 4.6 * モバイルアプリケーションのパッケージ化とオンラインストアへの書き出し](http://help.adobe.com/ja_JP/flex/mobileapps/WS4bebcd66a74275c3-416562ab12ee52291fa-8000.html)
* [ネイティブへ拡張し続けるAIRは“3”でどうなるのか - ＠IT](http://www.atmarkit.co.jp/fsmart/articles/air_int/01.html)

### Delphi XE

* [Delphi XE と FireMonkey について教えてもらった - Togetter](http://togetter.com/li/598248)
* [エンバカデロ技術文書ライブラリ](http://www.embarcadero.com/jp/technical-papers-japanese) の 「ホワイトペーパー: モバイル開発のためのDelphi言語」

### PhoneGap

* [PhoneGap API Documentation](http://docs.phonegap.com/en/3.2.0/index.html)

### Qt Mobile

* [Qt Mobile Edition](http://qt.digia.com/Qtmobileedition)

## まとめ

言語はまあそれぞれ違いますよね当然。

実行モデルは、iOS はその規約上全て AOT(Titanium を除く)、事前コンパイル方式です。
Android の方は、JIT型が Xamarin と AIR、AOT型が DelphiXE 、Titanium は何ていうの、iOS でも Androide でもインタプリタ？

画面を構成するパーツについては、各PFのネイティブUIを使うのが Xamarin、Titanium で、開発ツール側で頑張って全部レンダリングするのが AIR と DelphiXE 。DelphiXE は "Pixel Perfect" という技術でネイティブと寸分たがわない見た目を実現してるそうです。

ネイティブ機能の呼び出しは各社、なんらかの手段を用意してます（そりゃそうだ

これらの仕組みの違いが、それぞれメリット・デメリットを生み出します。
「使用上の注意をよく読み用法、用量を守り正しくお使いください」と言いたいところですが「それならはじめからネイティブでやるよ！」と突っ込まれそうですね。

[Xamarin Advent Calendar](http://qiita.com/advent-calendar/2013/xamarin) で、Xamarin の特徴を一つでも伝えられたら、と思います。


以上、文字だらけの記事にお付き合い頂きありがとうございました。