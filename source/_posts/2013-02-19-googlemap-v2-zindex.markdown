---
layout: post
title: "Google Maps Android API v2 の図形の描画順(zIndex)を探る"
date: 2012-12-25 17:48
comments: true
categories: [Android, GoogleMapsAPI]
---
Google Maps Android API v1 では Overlay をレイヤのように使うことで図形群の前後関係をコントロールできましたが、v2 では `Polyline` や `Polygon` クラスに `zIndex` が導入されました。

いくつかの図形を重ねて描画し、zIndex が与える影響を調べてみました。

<!-- more -->

## zIndex なし の場合

まず zIndex を指定しない場合。

1. マーカー
2. 線(太)
3. 線(細)
4. ポリゴン(青)
5. ポリゴン(緑)

の順で ```GoogleMap``` に追加しています。

```java NoZindex.java
final LatLng TOKYO = new LatLng(35.691, 139.693);
final LatLng NAGOYA = new LatLng(35.1805, 136.9073);
final LatLng TOYOHASHI = new LatLng(34.770, 137.391);
final LatLng MATSUMOTO = new LatLng(36.239, 137.969);
final LatLng SHIZUOKA = new LatLng(34.99, 138.39);
final LatLng MAEBASHI = new LatLng(36.38, 139.04);

// マーカー
mMap.addMarker(new MarkerOptions()
	.position(new LatLng(35.47, 138.71))
	.title("富士山"));

// ライン
mMap.addPolyline(new PolylineOptions()
	.add(TOKYO, NAGOYA)
    .width(40)
    .color(Color.BLUE));
mMap.addPolyline(new PolylineOptions()
	.add(TOKYO, NAGOYA)
    .width(10)
    .color(Color.RED));

// ポリゴン
mMap.addPolygon(new PolygonOptions()
	.add(TOKYO, MATSUMOTO, TOYOHASHI)
	.fillColor(Color.CYAN));
mMap.addPolygon(new PolygonOptions()
	.add(TOKYO, MAEBASHI, SHIZUOKA)
	.fillColor(Color.GREEN));
```

結果、こうなりました。

![noZindex](https://dl.dropbox.com/u/264530/qiita/zindex_off.png)

前後関係を見ると、奥から

1. ポリゴン(青)
2. ポリゴン(緑)
3. 線(太)
4. 線(細)
5. マーカー

となりました。描画順＝追加した順、であれば、マーカーや線は、ポリゴンによって覆い隠されてしまうのですが、そうなりませんでした。
図形によって前後関係が決められていて、
奥から ポリゴン → ライン → マーカー となるようです。
ちなみに、zindex を指定しない時の既定値は ```0``` です。

## zIndex を設定してみる

次に zIndex を次のように設定してみます。

```java WithZindex.java
// マーカー
mMap.addMarker(new MarkerOptions()
	.position(new LatLng(35.47, 138.71))
	.title("富士山"));

// ライン
mMap.addPolyline(new PolylineOptions()
	.add(TOKYO, NAGOYA)
    .width(40)
    .color(Color.BLUE)
    .zIndex(1));
mMap.addPolyline(new PolylineOptions()
	.add(TOKYO, NAGOYA)
    .width(10)
    .color(Color.RED)
    .zIndex(2));

// ポリゴン
mMap.addPolygon(new PolygonOptions()
	.add(TOKYO, MATSUMOTO, TOYOHASHI)
	.fillColor(Color.CYAN)
	.zIndex(3));
mMap.addPolygon(new PolygonOptions()
	.add(TOKYO, MAEBASHI, SHIZUOKA)
	.fillColor(Color.GREEN)
	.zIndex(4));
```

結果は以下のとおり。

![noZindex](https://dl.dropbox.com/u/264530/qiita/zindex_on.png)

1. 線(太)
2. 線(細)
3. ポリゴン(青)
4. ポリゴン(緑)
5. マーカー

む、ライン、ポリゴンに関係なく、zIndex で指定した順に奥から描画されているようです。

## Developer Guide の説明
いつも試してから見る公式ガイドｗ

* [Customize a Polyline - Google Maps Android API v2](https://developers.google.com/maps/documentation/android/lines#customize_a_polyline)

> Z-index
The stack order of this Polyline, relative to other overlays (polylines, polygons, ground overlays and tile overlays) in the map. A Polyline with a high z-index is drawn above overlays with lower z-indexes. Two overlays with the same z-index are drawn in an arbitrary order. Set this property with PolylineOptions.zIndex(). You can change this after the Polyline has been added to the map with the Polyline.setZIndex() method.

まあ、それっぽいことが書いてあるわｗ
どうやら `GroundOverlay` や `TileOverlay` にも zIndex があり、前後関係をコントロールできるようですね。

## まとめ
Google Maps Android API v2 の Polyline と Polygon の zIndex はまとめると次のようになります。

* 指定しない(```0```)と、追加の順番に関わらず ***Polyline の方が Polygon よりも手前*** に描画される。Polyline 同士、Polygon 同士は、追加順に奥から描画される。
* 指定した場合、Polyline、Polygon の区別なく、***zIndex の順で*** 奥から描画される。
* マーカーは常に一番手前に描画される。

ということで、zIndex をうまく設定すれば、図形群の前後関係を調整・変更することができます。