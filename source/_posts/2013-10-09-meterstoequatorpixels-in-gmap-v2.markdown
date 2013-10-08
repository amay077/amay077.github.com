---
layout: post
title: "metersToEquatorPixels を Google Maps Android API v2 で"
date: 2013-10-09 00:21
comments: true
categories: [Android, Java, Geo, GoogleMapsAPI]
---
Google Map API v1 には、「地図上の距離(ｍ)を画面上の距離(ピクセル)に変換する」ためのメソッド [Projection.metersToEquatorPixels](https://developers.google.com/maps/documentation/android/v1/reference/com/google/android/maps/Projection#metersToEquatorPixels(float)) がありましたが、v2 ではなくなってしまいました。
<!--more-->
ので、以下のような代替関数を作ってみました。

```java metersToEquatorPixels.java
public static int metersToEquatorPixels(GoogleMap map, LatLng base, float meters) {
	final double OFFSET_LON = 0.5d;
	
	Location baseLoc = new Location("");
	baseLoc.setLatitude(base.latitude);
	baseLoc.setLongitude(base.longitude);
	
	Location dest = new Location("");
	dest.setLatitude(base.latitude);
	dest.setLongitude(base.longitude + OFFSET_LON);
	
	double degPerMeter = OFFSET_LON / baseLoc.distanceTo(dest); // 1m は何度？
	double lonDistance = meters * degPerMeter; // m を度に変換

	Projection proj = map.getProjection();
	Point basePt = proj.toScreenLocation(base);
	Point destPt = proj.toScreenLocation(new LatLng(base.latitude, base.longitude + lonDistance));
	
	return Math.abs(destPt.x - basePt.x);
}
```

行っていることは単純で、基準となる緯度経度:``base`` から、適当に(ここでは 0.5度)東へ移動した緯度経度を ``Location.distanceTo`` で求め、その結果から、「1ｍは何度か？」を求めます。あとは、この係数を使って 地図上の距離:``meters`` を度に変換し、最後に、``base`` と移動後の緯度経度それぞれを画面座標に変換して、画面上の距離を返す、というものです。

「1ｍは何度か？」は、赤道上の値を使っても良いのですが、緯度によって値が大きく変わるので、このような手法を取りました。

ただこれでも、求める距離の精度によっては、``OFFSET_LON`` の値の調整が必要な気がします。また、経度:0 をまたぐような地域では正しく動かない気がします。(いずれも未検証)

また、緯度方向にもそれなりに正確な数値を出すには、上記と同じことを緯度に対しても行う必要があります。(これは v1 の API にもなかった)

v2 になって、描画系でピクセル座標を意識することはなくなったんであまり使うことも無いと思いますが、なにかで必要になったら思い出す程度で。

### 追記

あとで気づいたんですが、 v1 の [Projection.metersToEquatorPixels](https://developers.google.com/maps/documentation/android/v1/reference/com/google/android/maps/Projection#metersToEquatorPixels(float)) は、赤道上の距離で算出してたんですね。それと比べるとちょっとオーバースペックでした。

 それと、この記事を書く前に私のツイートを読まれた @honjo2 さんが、 v1 と同じ(赤道の距離を使う)仕様の関数を公開してくださいました。

<blockquote class="twitter-tweet"><p>どうぞ <a href="https://t.co/quYnqvn1tw">https://t.co/quYnqvn1tw</a> RT <a href="https://twitter.com/amay077">@amay077</a>: Google Map Android v2 になって metersToEquatorPixels がなくなっちゃったのが地味に不便だ。</p>&mdash; 本城 博昭 (@honjo2) <a href="https://twitter.com/honjo2/statuses/387368608541589505">October 8, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>