---
layout: post
title: "爆速エミュレータで Google Maps Android API v2 を動かす"
date: 2013-10-20 21:58
comments: true
categories: [Android, GoogleMapsAPI, Geo]
---
Google APIs のエミュレータで Google Map Android API v2 が動くようになったのですが、やっぱり遅い、使えん。
という訳で、Intel の爆速エミュレータで GMapV2 を使う手順の備忘録です。(ご利用は自己責任で)
<!--more-->
## 爆速エミュレータの導入

こちら等を参考にセットアップします。

* [Android再入門 - エミュレータの作成](http://qiita.com/gabu/items/8bc1a11f1382409f1d2a)

## エミュレータで Google Maps Android API v2 を動かす

``com.google.android.gms.apk`` と ``com.android.vending.apk`` が必要なのでどうにかして入手しインストールします。(ほとんど答えだけど下記参照)

* [Running Google Maps v2 on Android Emulator - Stack Overflow](http://stackoverflow.com/questions/14040185/running-google-maps-v2-on-android-emulator)

apk が古いとうまく動作しません。なるべく新しいものを探しましょう。(ﾎﾞｿ

## AndroidManifest.xml から com.google.android.maps の定義を消す

意外とハマったのがコレ。

``AndroidManifest.xml`` で

```
<uses-library android:name="com.google.android.maps" />
```

が定義してあると、

```
10-20 11:35:52.977: E/PackageManager(1178): Package xxxx requires unavailable shared library com.google.android.maps; failing!
```

というエラーになります。

この ``com.google.android.maps`` は Google Maps API v1 で必要だったもので、v2 では必要ありません。削除しましょう。

v2 の使い方を説明するブログやサイトで、これが含まれてしまってるものがあるようです。(かくいう自分もそんなサイトからコピペしてきたまま使ってたのでエラーになりました（汗）

## 動かす

あとは、実機と変わりません。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_gmapv2_on_intel_emulator_01.png)

やばい、PC性能とネットワーク環境のおかげで実機より快適になったｗ

## 参考

* [AndroidでGoogle Maps v2 をエミュレータで動かしてしかも爆速 - リア充爆発日記](http://d.hatena.ne.jp/ria10/20121218/1355794748)
* [エミュレータでGoogle Maps for Android V2を動かす方法 | アプリ開発とRaspberry PIとArduino実験](http://denshikousaku.net/how-to-make-android-google-maps-v2-work-in-android-emulator)
* [Running Google Maps v2 on Android Emulator - Stack Overflow](http://stackoverflow.com/questions/14040185/running-google-maps-v2-on-android-emulator)
