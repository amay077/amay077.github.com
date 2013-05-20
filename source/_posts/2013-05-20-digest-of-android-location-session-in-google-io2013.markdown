---
layout: post
title: "Google I/O 2013 の Android Location セッションまとめ"
date: 2013-05-20 15:13
comments: true
categories: [Android, GoogleMapsAPI]
---
これは、[Google I/O 2013 - Beyond the Blue Dot: New Features in Android Location - YouTube](http://www.youtube.com/watch?v=URcVZybzMUI) から主要な部分を切り出して勝手な解説を加えたものです。
<!--more-->
## 時間がない人のために

このセッションは、Android に追加された「Fused Location Provider」「Geofencing」「Activity Recognition」に関するお話です。

これらの３つのデモだけ見れば、だいたい「すげー」ってなるかと。

* [Fused Location Provider のデモ](http://www.youtube.com/watch?feature=player_detailpage&v=URcVZybzMUI#t=733s)

* [Geofencing のデモ](http://www.youtube.com/watch?feature=player_detailpage&v=URcVZybzMUI#t=1195s)

* [Activity Recognition のデモ](http://www.youtube.com/watch?feature=player_detailpage&v=URcVZybzMUI#t=1661s)

で、これらの機能は Google Play services として提供されるので、新しい端末を待たなくても**今日から使えます**よ、と。


## はじめに

タイトルは「Beyond the Blue Dot」、Blue Dot とは…デモを観たら分かります。

!["1"](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_001.png)

Intro&Closing 担当の Waleed さん(右)と、Deep Dive してくれる Jaikumar さん(左)

![2](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_002.png)

基本として、各種測位技術の比較表を。

![8](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_008.png)


## Fused Location Provider

Intro を早々に飛ばして、ここから各機能の詳細説明。

まず、現在の Location 周りの構成図。これが…

![9](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_009.png)

こうなる。今までのレイヤの上に Google Play-services として構成される。

![17](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_017.png)

さあてお待ちかねのデモ。

まずは、実際の経路(緑) vs GPS(黄色)

画像だけだと若干分かりづらいけど、屋内はすっ飛ばされてる。

![18](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_018.png)

次、緑 vs WiFi測位(赤)

もう、カックカクなのであります。

![19](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_019.png)

そして、緑 vs Fused Location Provider(青 the Blue Dot)

屋外は GPS によるスムースな軌跡、屋内は WiFi＋センサーによる自律測位で十分に滑らか。入出、退出時の切り替えも自動で行われる。(拍手！)

![20](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_020.png)

使い方。

[LocationClient](http://developer.android.com/reference/com/google/android/gms/location/LocationClient.html) というのが増えてるので、それを使います。

``connect`` した後は、``LocationListener`` が使えます。(注:``android.location.LocationListener`` ではなく新しい ``com.google.android.gms.location.LocationListener`` でした)

![21](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_021.png)

![22](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_022.png)

[setPriority](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#setPriority(int) で電池消費と精度をコントロールできるとのこと。

![24](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_024.png)

## Geofencing

次のトピック、ジオフェンシング。

さっそくデモから。

ジオフェンスが２つ仕掛けてあって、自車がフェンス内に入ると色が変わるというもの。

![26](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_026.png)

使い方。

同じく ``LocationClient`` から。
結果は IntentService で受け取ります。

![28](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_028.png)

そして嬉しいのがコレ。

消費電力が今までの 1/3 になってるとのことです。
実は、ジオフェンシングの機能自体は [addProximityAlert](http://developer.android.com/reference/android/location/LocationManager.html#addProximityAlert(double, double, float, long, android.app.PendingIntent) という形で既存だったのです(存在は知ってたが使ったこと無い)。

ユーザーの大雑把な場所や、現在の状態(歩いてるのか留まっているのか)やら、ハードウェアに直接処理させているので実現できた、とか言ってるみたいです。

![30](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_030.png)

## Activity Recognition

最後のトピック、行動(状態)認識。

乗り物、徒歩、留まってる、自転車 を判別できます。

![32](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_032.png)

これはライブデモ。
自転車と認識されているのは…

![40](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_040.png)

…マークさんの実演でしたー。(会場ここが一番盛り上がってた)

![41](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_041.png)

そして使い方、
[ActivityRecognitionClient](http://developer.android.com/reference/com/google/android/gms/location/ActivityRecognitionClient.html) というのを使います。
これは早速使ってみたのでこちらもご参考に。

* [Google I/O 2013 で発表された行動認識(Activity Recognition)を使ってみる - Experiments Never Fail](http://amay077.github.io/blog/2013/05/18/getting-started-activity-recognition/)

![43](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_043.png)

![44](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_044.png)

## まとめ

Waleed さんに戻ってまとめなど。

Google Play-services で提供してるから、使うなら？「今でしょ！」(言ってません)。
これからも、消費電力・精度・使える場所(?) を改良していくぜい、とのこと。

![49](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_049.png)

どうもありがとうございました。

![50](https://dl.dropboxusercontent.com/u/264530/qiita/google_io_android_location_050.png)


## 雑感

Fused Location Provider(GPS+WiFi+自律測位)は、カーナビ(GPS+自律測位)では普通に行われているものの、あちらは道路の上という縛りがあるのに対し、こちらはフリーダムな移動を処理しなければならないのですが、実用レベルの API を、誰でも使える形で提供してくる Google さん怖すぎ。

国内でも大学とかベンチャーが頑張って屋内測位とか行動認識技術開発してるので、ぜひがんばってください。

「なぜ Google Play-services で提供するのか？」に:

* 既存デバイスにも対応できる
* 短いサイクルでリリースできる

などのメリットを説明していましたが、その後ツイッターで、

<blockquote class="twitter-tweet" data-conversation="none" lang="ja"><p>@<a href="https://twitter.com/amay077">amay077</a> おおー、参考になりました。ありがとうございます。ところで、これGooglePlayクラスのしただったのね。業務用のタブレットやスマホだとGooglePlay使えないから、ちょっと残念。</p>&mdash; new hirofumi hayashiさん (@picaosgeo) <a href="https://twitter.com/picaosgeo/status/335587102043549696">2013年5月18日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

というやりとりをして気づきました。Google Play-services って「(オープンソースとしての)Android」じゃないのねーと(まあこれほどの技術の内部を公開なんて普通に考えてもありえない話ですが)。

**「Google Play-services が使えない端末では利用できない」** というのは割と盲点になりそうです(Kindle もダメだよね、たぶん。しかしエミュレータですら使えないのは何とかして欲しい)。

[Chrome との統合がうわさされる Android](http://itpro.nikkeibp.co.jp/article/COLUMN/20130321/464924/) ですが、Android の名を冠していない Google Play-services が充実していくのは、その流れなのかも知れませんね。