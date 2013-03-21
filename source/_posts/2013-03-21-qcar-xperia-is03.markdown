---
layout: post
title: "クアルコムの AR SDK ”QCAR” を Xperia と IS03 で動かしてみた"
date: 2010-12-1 2:32
comments: true
categories: [Android, Java, AR, QCAR]
---
QCAR とは クアルコム社 が[提供している](http://japanese.engadget.com/2010/10/07/android-ar-sdk/) Android 用の AR（拡張現実） SDK です。
<!--more-->

[Qualcomm Developper Network - AR SDK | Qualcomm Augmented Reality](https://ar.qualcomm.com/qdevnet/sdk)

こちらの方々

* クアルコムのAndroid向けAR SDK（QCAR）を動かしてみた - kotamzの日記
* QualcommのAR SDK "QCAR" で遊んでみた - hdk_embeddedの日記

が使い方を説明してくださってますので、自分も（仕事で）やってみました。

対応している端末は Desire , Nexus One との事でしたが、どちらも持ってないので、手持ちの Xperia (※ Android 2.1 に上げたやつ) と IS03 で試してみました。

まず Xperia 。

お、できた！（QCAR というより NDK と Cygwin でかなりてこずりましたが、順番にやれば大丈夫）

!["1"](https://dl.dropbox.com/u/264530/qiita/qcar_xperia_1.png)
!["2"](https://dl.dropbox.com/u/264530/qiita/qcar_xperia_2.png)

なかなかイイ感じに認識します。

ただ、MultiImageTargets だけはうまく認識してくれませんでした。せっかく立方体作ったのに…）
カメラのピントなどを調節すれば認識するのかも知れません。

(2010.12.2追記)あ、動いた。なかなか認識してくれなかったけど、正面（ライオンがついてない方）をまっすぐ映してじっとしてたらできました。

!["3"](https://dl.dropbox.com/u/264530/qiita/qcar_xperia_3.png)

ミンティアの上にポットを載せてみました(^ ^)

!["4"](https://dl.dropbox.com/u/264530/qiita/qcar_xperia_4.png)
 
自分の画像を使うには、Qualcomm Developper Network の AR SDK のページ内にある [My Trackables](https://ar.qualcomm.com/qdevnet/projects) へ行き、プロジェクトを作った後、画像（JPG か PNG で 2MB未満）をアップロードします。

!["5"](https://dl.dropbox.com/u/264530/qiita/qcar_xperia_5.png)

アップロードできると↑こんな画面になります。たぶん ☆ が多い方が認識しやすい画像です。（試しにホットペッパーの見開き２ページをアップしたら ☆ がひとつも付きませんでした。→追記(2010.12.2)ちゃんとスキャンしてもらってグレイスケールに変換して制限2MBギリギリな画像でアップしたら、☆が4つ付きました。）

このデータをアプリで使うには、項目をチェックして download selected trackables をクリックすると ZIP がダウンロードできるので、その中身を ImageTargets サンプル の assets フォルダに上書きしてあげると適用できます。（たぶん、複数項目をチェックしてダウンロードすると画像によってポットの色が変わるようになると思います。→追記(2010.12.2)大嘘。JNI 側の ImageTargets.cpp をダウンロードした画像の名前（config.xml を見ると分かる）に合わせて修正してあげないとダメっぽい）

自分で動かすと相当オモシロイので、是非お試しください！ 

次に IS03 ですが、、、動きませんでしたー(^ ^;

エラーは出ませんが、カメラが起動しないです。
ソースコードを直すと動くかも知れないので、また調べたいと思います。

今日はこんなところで。

### 追記（2010.12.2）：Xperia ではバグがあるらしい

QCAR SDK のページに 「Xperia で動かすとバグがある」的な情報が載ってました。MultiImageTargets サンプルがうまく動いてくれなかったのはこの為かなあ？

> Note: We have currently identified a bug that prevents the SDK from working correctly on the Sony Ericsson Xperia X10i (QSD 8250) running Android 2.1. This page will be updated once it is fixed.
>
> via [AR SDK | Qualcomm Augmented Realit](https://ar.qualcomm.com/qdevnet/sdk)

### 追記（2010.12.2）：認識できる画像について

* アルゴリズムは良く分からないけど、画像から特徴点を算出している。これに色は関係ないみたい。カラー画像もアップしたらグレイスケールに変換されてた。
* 認識率のよい（☆がいっぱい付く）画像は、ごちゃごちゃしている画像。
* ホットペッパーの各ページは、構図は似ているけど、店の写真がそれぞれ違うので認識率は高い。
* 店内写真だけ切り抜いてアップしたら、写真によってバラつきがある。下の左の写真は認識率が高いが、右は低い。

!["6"](https://dl.dropbox.com/u/264530/qiita/qcar_xperia_6.png)
!["7"](https://dl.dropbox.com/u/264530/qiita/qcar_xperia_7.png)
