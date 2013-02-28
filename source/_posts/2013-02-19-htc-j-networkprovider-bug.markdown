---
layout: post
title: "HTC J で NETWORK_PROVIDER で位置測位した時の機種依存バグ？"
date: 2013-02-19 00:41
comments: true
categories: [Android]
---
[Androidの機種依存問題を吸収するプロジェクトAndroid-Device-Compatibilityを公開したお話](mixi Engineers' Blog http://alpha.mixi.co.jp/2013/11572/) を見て、あーそう言えばこんなんあったなーと思い出したのでメモ。

## 症状
下記条件を満たす時に、LocationManager.requestLocationUpdates すると、最新の位置が取得されず、**前回測位した位置** が返される。

時刻は更新されてたか、、、あーどうだったかな。

<!-- more -->

## 条件

* NETWORK_PROVIDER で位置を取得した場合（GPS じゃない）
* WiFi が無効の場合（つまり携帯電話の基地局のみを使った測位の場合）
* スリープ中な場合
* HTC J である(IS03, Nexus S では発生してない。 HTC J Butterfly はわからない)

## こんな事してたら見つけた
早い話が HexRinger なんですが、このアプリは、AlarmManager で一定時間毎に WiFi/基地局から位置を取得しています。デバッグで移動中のログを取ってたら、「この時間帯、電車で寝てたのになんか位置変わってないんだけどｗ」となりました。

## 対策
WakeLock する。ただし PARTIAL_WAKE_LOCK だとダメで、画面も起こしてやらないとダメだった。
SCREEN あるいは FULL_WAKELOCK が必要な常駐アプリって、最悪やん。

## いろいろ試した事とか推測
* 測位前に通信させればいけるか？と思ったけどダメだった。
* そもそも「最寄りの基地局情報」がスリープ状態だと更新されないんじゃないか説。


設置場所が山奥過ぎてこの地雷を踏む人はそうはいないと思いますが、「HTC J だけど問題ないよ？」「Butterfly でもダメだったわー」とかの情報あったらコメントください。 m(_ _)m