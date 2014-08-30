---
layout: post
title: "Xamarin Studio の iOSデザイナでは、Unwind Segue に Identifier が付けられないようです"
date: 2014-08-31 00:48:46 +0900
comments: true
categories: [Xamarin, iOS, Xcode]
---
* [Xamarin.iOSでの画面遷移で「戻る」を実現するには？（Storyboard、Unwind Segue使用） - Build Insider](http://www.buildinsider.net/mobile/xamarintips/0016)

に Unwind Segue について書きましたが、載せきれなかったことを補足します。
<!--more-->
上記Tipsでは、Unwind Segueを作るのに、ボタンとExitを結んでますが、``PerformSegue``で Unwind Segue を使いたい場合もありましょう。

``PerformsSegue``に渡す Identifier はどこで付与するの？とあちこち探しましたが、どうも現在の Xamarin Studio（のiOSデザイナ）はまだ対応していないようです。というか、そもそもコントロールから引っ張るタイプのUnwind Segueしか作れないみたいです。

Xcode の Interface Builder では 

* [Technical Note TN2298: Using Unwind Segues](https://developer.apple.com/library/ios/technotes/tn2298/_index.html#//apple_ref/doc/uid/DTS40013591-CH1-TNTAG2-ADDING_AN_UNWIND_SEGUE_TO_A_STORYBOARD)

にあるように、「ViewControllerのアイコンをExitアイコンへControl+ドラッグ＆ドロップする」で、”Manual” Unwind Segue が作成できるのですが、Xamarin Studio では、それっぽい操作をしても反応がありません。

当然ながら、Xcode で作成した Segue は Xamarin Studio でも使えるので、Manual Unwind Segue の作成と Identifier の付与は Xcode で行って、Xamarin側で ``PerformSegue`` を呼び出せばいいと思います。
