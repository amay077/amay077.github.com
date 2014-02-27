---
layout: post
title: "Xamarin によるクロスプラットフォームモバイルアプリ開発、資料と補足"
date: 2014-02-27 21:40
comments: true
categories: [Xamarin, Android, iOS, C#]
---
2/26 の [うずらインキュベータ](http://atnd.org/events/47898) という勉強会で、Xamarin の話をしました。
<!--more-->

<iframe src="http://www.slideshare.net/slideshow/embed_code/31674661" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px 1px 0; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/amay077/xamarin-31674661" title="Xamarin によるクロスプラットフォームモバイルアプリ開発" target="_blank">Xamarin によるクロスプラットフォームモバイルアプリ開発</a> </strong> from <strong><a href="http://www.slideshare.net/amay077" target="_blank">amay 077</a></strong> </div>

45分という長い時間話すのは勉強会では初めてだったのですが、なんとか説明し切ることができました。（ちょっとデモが中途半端になってしまいましたが）

資料は [Qiita に書いてきた](http://qiita.com/tags/xamarin) 内容のまとめみたいなものですが、少し補足します。

## Xamarin で作った経験あるの？

仕事では、まだ無いです（^_^;）
個人アプリでは「[富士フォト](https://itunes.apple.com/us/app/fu-shifoto/id806913229)」というのを iOS 用は Xamarin.iOS で作りました。[Android](https://play.google.com/store/apps/details?id=com.amay077.android.fujiphoto) は Java ですが Xamarin 化したいな。

## Win+Visual Studio ではダメなの？

個人の見解ですから（^_^;）
私も元々は Windowsの開発がメインで Visual Studio の強力さは知っていますが、iOS やるならどういう形にせよ Mac+Xcode を扱わないといけないので、慣れておいた方がよいかなと。

また、Microsoft との提携以降、Microsoft のエバンジェリストさんや MVP の方々が Visual Studio + Xamarin の話をものすごく展開されているので、そちらにお任せした次第です。

## 実行モデルのとこ

JavaSE が .NET に置き換わる図になっていますが、実際には少し違っていて、JavaSE のラッパもあります。例えば文字列型には、``System.String`` と ``Java.Lang.String`` があります。当然、理由がなければ前者を使った方がよいわけですが。

iOS のスタックに関しては、実はどこからどこまでが「CocoaTouch」なのかよく分かってません。

## 他のクロスプラットフォーム開発ツールとの比較

Titanium, AIR については2年くらい前に少し触ったことがあります。PhoneGap と DelphiXE についてはスペックを見て＆詳しい方からの情報を元にしてます。

Titanium は次期 [Ti.Next](http://titanium-mobile.jp/38) では JavaScriptCore を使ってすんごく速くなるそうですし、AIR も当時よりだいぶ [高速になったらしい](http://www.slideshare.net/pik256/dev-sumi2014-13c4rev) です。

## C# のとこ

https://xamarin.com/csharp よりは悪意のないコードかとｗ
（Objective-C と比較しようとすると Obj-C の方が画面に入らないので Java との比較にしました。）



こんなところで。
何かおかしなところがあったらコメント頂けるとありがたいです。
