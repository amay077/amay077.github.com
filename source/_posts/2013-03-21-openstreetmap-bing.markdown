---
layout: post
title: "OpenStreetMap で Bing の衛星写真が使えるようになった！けどズレが気になる…"
date: 2010-12-1 7:29
comments: true
categories: [OpenStreetMap, Map]
---
[OpenStreetMap のエラい人が Microsoft に加入した](http://journal.mycom.co.jp/news/2010/11/26/078/)事を受け、OpenStreetMap（以下 OSM） の背景地図に Bing Map の衛星写真が重ねられるようになりました。
<!--more-->

* [JA:Bing - OpenStreetMap Wiki](http://wiki.openstreetmap.org/wiki/JA:Bing)

これで、鮮明な衛星写真を元に GPSログ がなくても地図が描けるようになりましたね。

って少し気になることが。

 

たしか Googleマップ もそうだったですけど、衛生写真 と通常の地図（ベクトル地図）で少しズレてません？

というわけで Googleマップ と、OSM で私が最近ロギングして描いた地図で比較してみました。

!["1"](https://dl.dropbox.com/u/264530/qiita/osm_bing_1.png)
!["2"](https://dl.dropbox.com/u/264530/qiita/osm_bing_2.png)

一つ目が Google 、 二つ目が OSM (Bing) です。
 

やっぱりズレてますねー。

Googleマップ も Bing マップもズレの量はともかく、同じ方向にズレている感じです。

地点によっては、まったくズレていない所もあるし、画像補正の関係でこうなっちゃうのかな…。

 

ということで、OSM に Bing のチカラが加わったのは嬉しいのですが、精度が要求される所では、やっぱり GPS ログを収集しましょう、ということでご注意ください。

 

それにしても加入して即、Bing を OSM に持ち込むなんてスゴいですね。まるで メカドックに那智さんが加入して即、サバンナRX-7 をスリーローターに改造しちゃうみたいな。。。(わかるひといるかなー？)