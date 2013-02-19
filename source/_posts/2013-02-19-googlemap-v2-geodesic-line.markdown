---
layout: post
title: "Google Maps Android API v2 で大圏コースを表示する"
date: 2012-12-24 17:51
comments: true
categories: [Android, GoogleMapsAPI]
---
GoogleMap は[メルカトル図法](http://ja.wikipedia.org/wiki/%E3%83%A1%E3%83%AB%E3%82%AB%E3%83%88%E3%83%AB%E5%9B%B3%E6%B3%95)なので、地図上の２点間を結んだ直線は最短距離になりません。(メルカトル図法で正しいのは角度だけ、でしたよね。)

地球上の２点間の最短距離は「[大圏コース](http://ja.wikipedia.org/wiki/%E5%A4%A7%E5%9C%8F%E3%82%B3%E3%83%BC%E3%82%B9)」と呼ばれます。

で、Android 版の新しい API を使うと、この大圏コースを簡単に表示することができます。

<!-- more -->

こんな感じ、赤がただの直線、青が大圏コースです。

![greatcircle](https://dl.dropbox.com/u/264530/qiita/greatcircle.png)

やり方は以下のとおりで、```geodesic``` を ```true``` にすれば大圏コースになります。

```java GeodesicPolyline.java
GoogleMap mMap;

final LatLng TOKYO = new LatLng(35.691, 139.693);
final LatLng HAWAII = new LatLng(19.87, -155.56);

// ただの直線
mMap.addPolyline(new PolylineOptions()
	.add(TOKYO, HAWAII)
    .width(5)
    .color(Color.RED)
    .geodesic(false));

// 大圏コース
mMap.addPolyline(new PolylineOptions()
	.add(TOKYO, HAWAII)
    .width(5)
    .color(Color.BLUE)
    .geodesic(true));
```

Developper Guide に説明があります。

* [Polylines and Polygons - Google Maps Android API v2](https://developers.google.com/maps/documentation/android/lines#geodesic_and_non-geodesic_lines)

> A Geodesic line is a line that follows the curvature of the earth. In contrast, a non-geodesic line will be drawn using the coordinate system of your screen. By Default, Polyline and Polygon objects will draw non-geodesic lines. You can change any Polyline or Polygon to use geodesic lines by setting the geodesic property to true.

> 適当訳
> Geodesic なラインとは地球上の曲がった線のことです。non-geodesic ってやつは画面座標系で描いた線で、これがデフォルトです。Polyline や Polygon を geodesic にしたければ geodesic プロパティを true にするとよろし。

ちなみにこの「２点間の最短距離」は [Location.distanceBetween](http://developer.android.com/reference/android/location/Location.html#distanceBetween(double, double, double, double, float[]) メソッドで求められます。

どうやら、Javascript 版にはもともとこの機能があったようで、以下のサイトに例があります。

* [Googleマップで大圏コースを表示する](http://user.numazu-ct.ac.jp/~tsato/tsato/geoweb/googlemaps/great-circle/)

あと、iOS 版の Google Maps API にもあるみた…あれ、ないや。
(余談ですが iOS 版の API には Polygon を描く機能もないんですね)
 
* [Polylines - Google Maps SDK for iOS](https://developers.google.com/maps/documentation/ios/lines)

## その他
つか、Android の GoogleMap さん、思いっきり縮小しても世界が画面内に収まらないので、大圏コースの例が見せにくいわ。