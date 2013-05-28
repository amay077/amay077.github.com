---
layout: post
title: "アプリ内でブロードキャストする場合は LocalBroadcastManager を使うと良いらしい"
date: 2013-05-28 19:25
comments: true
categories: [Android]
---
サポートパッケージに``LocalBroadcastManager`` というクラスがあるのを今頃知りまして。
<!--more-->
* [LocalBroadcastManager | Android Developers](http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)

ブロードキャストは使い方を誤るとデータを(アプリの)外部に流出させる可能性があるわけですが([Android アプリのセキュア設計・セキュアコーディングガイド](http://www.jssec.org/report/securecoding.html) 参照)、このクラスを使うと「他のアプリにデータを漏らさない」「意図しないブロードキャストを受信しない」「効率がよい」だそうです。

## いつ使うんですか？

たとえば GPS を使うアプリで、GPS の受信は ``IntentService`` にやらせて、受信した位置を地図に表示するために、IntentService からブロードキャスト投げて、``Activity`` に仕掛けたレシーバで受信する、なんてケースでしょうか。(いやそれは IntentService じゃなくて普通のサービスで aidl 使ってやれよ、とかいろいろあるわけですが。ん？PendingIntent でサービスを起動する方法だと、クライアントから bind するタイミングが無いからダメかな？)

## 使い方

stackoverflow に良い使い方が載ってました

* [android - how to use LocalBroadcastManager? - Stack Overflow](http://stackoverflow.com/a/8875292)

## 使ってみた

状態認識の結果も秘匿情報でしょう、ということで[以前](http://amay077.github.io/blog/2013/05/18/getting-started-activity-recognition/)作った ``ActivityRecognitionClient`` のサンプルを修正してみました。

* [amay077/androidactivityrecognizingsample · GitHub](https://github.com/amay077/androidactivityrecognizingsample/commit/a041b300d3e9fdfe6227c05c3f21fb1e3876bbad)

## まとめ

* ブロードキャストする時は、まず LocalBroadcastManager を使ってみよう。
* stackoverflow の回答へのダイレクトリンクとか、github の changeset へのダイレクトリンク便利すぎる！