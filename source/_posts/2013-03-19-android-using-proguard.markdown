---
layout: post
title: "Android で Proguard を使う(ADT17以降)"
date: 2013-03-19 21:45
comments: true
categories: [Android, Proguard]
---
割とハマったのでメモ。
<!--more-->

## Proguard.cfg を使う方法はもう古い

* [Androidに統合されたProGuardに関する改善点(ADT17) - Android(アンドロイド)情報-ブリリアントサービス](http://d.hatena.ne.jp/bs-android/20120325/1332662384)
更新日-2012/03/25

にあるように、ADT17 以降では、既定の難読化設定は SDK のディレクトリにありそれを参照するようになっている。
プロジェクト固有の設定は proguard-project.txt に書くように既定で準備されている。

Android proguard でググって出てくる情報は、大抵は Proguard.cfg として説明されているので、その情報は断片的には使えるだろうが、古い情報として注意して参照したほうがよい。


## -libraryjars について
外部ライブラリを使っている(/libs に jar を入れてる)場合、それらを -libraryjars xxx.jar と列挙しなければならないと **思っていた** が不要らしい。

-libraryjars に列挙したら最初の一つしか library jar として認識してくれなくて「なにこれ？」と思ってググったら、

* [Using ProGuard with Android - StackOverflow]
(http://stackoverflow.com/questions/11246842/using-proguard-with-android/11249755#11249755) 回答日-2012/06/28

>1) ProGuard manual > Troubleshooting > Note: duplicate definition of program/library class
>
>The Android Ant/Eclipse builds already specify -injars/-libraryjars for you. If you specify them again in your configuration, ProGuard notes that they are duplicated. So don't specify -injars/-libraryjars.

libs配下は既定で定義済だから proguard-project.txt の方で再定義しなくてよいらしい。

## -dontwarn とか -dontnote とか
* [Proguardでcan't find referenced class - そこはかとなく書くよ。](http://d.hatena.ne.jp/rudi/20110205/1296914415)

「ほんとかなぁ」に同意。warning潰すのがデファクトなツールで良いの？

### (私が)未解決な問題
とあるメソッドの引数 ``Hoge<Piyo> arg``  から、 ``Class#getGenericInterfaces()`` で ``Piyo`` を取得していたのだけど、Proguard かけたら、 ``Hoge<Piyo>`` が ``Hoge`` だけになって取れなくなってしまった。

Java のイレイジャ云々か？とも思ったけど、メソッドの引数ならいけるはずだし…

* [イレイジャではジェネリクスの何が消えるのか](http://blogs.wankuma.com/nagise/archive/2008/10/13/158708.aspx) 

StackOverflow さんによると、

* [Google Drive API doesn't play well with ProGuard (NPE)](http://stackoverflow.com/a/14449289/789062)

``-keepattributes *Annotation*`` をつけるといいよ的な事が書いてあるんだけど、うまくいかなかった。

## 雑感
情報が新旧錯綜してる発展途上なツールを苦労して使って、設定も実装に依存するし(自動化できない)、運用時もデバッグしづらいし、この難読化って作業、コスパどうなんすかね？ 所詮 Webサービスのフロントエンドである Java プログラムに、そこまで難読化にこだわることはないんじゃないかなーという私見です。

## 参考
* [ProGuardで-keepオプションのメモ - superdry memorandum :-D](http://d.hatena.ne.jp/Superdry/20110121/1295641171)