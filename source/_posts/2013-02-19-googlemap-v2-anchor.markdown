---
layout: post
title: "Google Maps API v2 のマーカーの Anchor を探る"
date: 2013-02-19 23:10
comments: true
categories: [Android, GoogleMapsAPI]
---

Google Maps API の Marker には、[Anchor](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/MarkerOptions#anchor(float, float) というプロパティがあり、緯度経度と、画像のどの位置をマッピングさせるかを設定する事ができます。

<!-- more -->

Marker の既定の画像は、よく見るピンみたいなやつですが、既定の Anchor は、0.5f/1.0f になっています。設定値は、画像に対する「x軸の割合」と「y軸の割合」で、言わんとすることは、「緯度経度の位置を画像の、横方向はちょうど真ん中、縦方向は最下部に合わせる」という事です。

![Marker.anchor の設定値](https://dl.dropbox.com/u/264530/qiita/marker_anchor.png)

以下のように、設定値を変更すると、それぞれマーカーの表示位置が変わります。(Android SDK に同梱される Google Maps API v2 のサンプルをベースにしています)

```java anchor_center_bottom.java
mAdelaide = mMap.addMarker(new MarkerOptions()
	.position(ADELAIDE)
    .title("Adelaide")
    .snippet("Population: 1,213,000")
    .anchor(0.5f, 1.0f)); // 既定値と同じ
```
```java anchor_left_top.java
mAdelaide = mMap.addMarker(new MarkerOptions()
	.position(ADELAIDE)
    .title("Adelaide")
    .snippet("Population: 1,213,000")
    .anchor(0.0f, 0.0f)); // 左上
```
```java anchor_right_middle.java
mAdelaide = mMap.addMarker(new MarkerOptions()
	.position(ADELAIDE)
    .title("Adelaide")
    .snippet("Population: 1,213,000")
    .anchor(1.0f, 0.5f)); // 右中
```

![Marker.anchor examples](https://dl.dropbox.com/u/264530/qiita/marker_anchor_sample.png)

## v1 ではどうだったっけ？
Google Maps API v1 では、[ItemizedOverlay](https://developers.google.com/maps/documentation/android/v1/reference/) の ``boundCenter`` と ``boundCenterBottom`` に相当する機能ですね。こちらはメソッド名の通り、「中央/中心」と「中央/下部」しか対応してなかったので、より柔軟になったと言えます。

## ハマりどころ
* 最初、anchor はピクセル指定だ、と勝手に勘違いして、画像のサイズを取得してゴニョゴニョやってたのは内緒。
* 0.0〜1.0 の範囲であることを忘れて、0〜100 の値を設定すると、**「マーカーが全部消えます」**