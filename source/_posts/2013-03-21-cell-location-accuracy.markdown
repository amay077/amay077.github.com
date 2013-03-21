---
layout: post
title: "基地局での位置測位の精度を調べてみた"
date: 2010-12-29 7:18
comments: true
categories: [Android, Gps, Map]
---
前回、WiFi 位置測位の精度を調べてみた の続きで、GPS も WiFi も無効だった場合、つまり携帯電話の基地局による位置測位の精度を調べてみました。
<!--more-->

経路は前回と同じです。
計測方法も同じで、ただ GPS も WiFi も端末の設定で OFF にしています。
で結果です。（青が 前回と同じGPSデータ、黄色が基地局の測位結果です）

!["photo"](https://dl.dropbox.com/u/264530/qiita/celllocation_map.png)

どうです？こちらも結構使えそうでしょ？
Accuracy （精度）の値は、3000～5000ｍ 、だいたい目視でもその範囲内で収まってる感じです。
しかも WiFi のように突然位置がぶっ飛ぶ現象も起こってません。

さて、んじゃあ WiFi 捨ててこっち使えば？あるいは WiFi とこっちで精度の高いほう使えば？ってトコなんですが、「WiFi が有効な時に、あえて基地局だけを使って測位する方法」が分からんのです。(・・;)

Android の位置を取得するための Provider は GPS_PROVIDER と NETWORK_PROVIDER の二つしかなく、WiFi と基地局は NETWORK_PROVIDER に分類されると思うのですが、その中で基地局のみと指定する事ができるのかすら分かってません。

誰か知ってたら教えてください。m(_ _)m