---
layout: post
title: "Google Maps Android API v2 の v1 からの変更点メモ"
date: 2012-12-20 16:10
comments: true
categories: [Android, GoogleMapsAPI]
---
[Android の Map API が刷新](http://www.atmarkit.co.jp/ait/articles/1212/04/news110.html)され、

* ベクトル地図！
* 回転、視点変更
* 屋内、地下街マップの表示

が可能になりました。Googleマップアプリの機能が API で提供された感じです。これは感涙。

v2 になって導入手順が大幅に変わりましたが、 [@adamrocker](https://twitter.com/adamrocker) さんが早速解説して下さっています。

* [throw Life - Google Maps Android API v2を使ってみた](http://www.adamrocker.com/blog/334/google-maps-android-api-v2.html)

ここでは、Google Map Android API v1 からの変更点を中心に紹介してみます。

## 互換性
***あ・り・ま・せ・ん、以上！***

Fragment 化、3D対応などにより、API の設計思想そのものが変わった様で、既存のコードをそのまま流用できることはないでしょう。
前の MapView に似せた [com.google.android.gms.maps.MapView](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/MapView) というクラスがありますが、クラス名が同じだけでメンバは全然別物です。

## じゃあ、v1 のあの機能は v2 でどうやんの？

### 移動、拡大・縮小など

![BasicMap](http://f.cl.ly/items/2d3M3T1v3W3C0H102d3L/gmap2_basic.png)

v1 では [MapController](https://developers.google.com/maps/documentation/android/v1/reference/com/google/android/maps/MapController.html) の animateTo とか setZoom で行なっていましたが、そもそも v2 には MapController クラスがなくなっています。

v2 では [GoogleMap](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/GoogleMap) クラスの animateCamera() や moveCamera() で移動、拡大・縮小を行います。
これらに加えて、角度(bearing)、視点(tilt) も設定します。animateCamera は Google Earth みたいなアニメーションが定義できてカッコいいですよ。

ちなみに、無段階ズームになった影響で、Zoom の値が int から float になっています。値が示す地図縮尺は変化ないようです。v1 の Zoom=15 と v2 の Zoom=15.0f は同じ意味です。

#### 参考
SDK に同梱されるサンプル /extras/google/google_play_services/samples/maps の該当箇所を添えておきます。

* CameraDemoActivity.java

### ズームボタンとかの表示ON/OFF

![UISettings](http://cl.ly/image/40220F3I163f/gmap2_uisettings.png)

v1 では MapView の setBuiltInZoomControls でズームボタンの表示ON/OFF ができました。
v2 では、GoogleMap から [UISettings](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/GoogleMap#getUiSettings() を取得して設定します。
v2 では、ズームボタンの他に、コンパスが増えているのでそれらの表示制御と、各種ジェスチャ(スクロール、回転、チルト)の有効/無効 が設定できます。

#### 参考

* UiSettingsDemoActivity.java

### マーカー表示
![Markers](http://cl.ly/image/2a0b2C073d1f/gmap2_markers.png)
API の使い方で一番大きく変わったのはこの辺りかと思います。

v1 では、MapView に [ItemizedOverlay](https://developers.google.com/maps/documentation/android/v1/reference/com/google/android/maps/ItemizedOverlay.html) を add して、OverlayItem(＝マーカー)を登録して…って感じでしたが、全滅です。

v2 でのマーカー表示は [GoogleMap.addMarker](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/GoogleMap.html#addMarker(com.google.android.gms.maps.model.MarkerOptions) を使います。
Overlay の概念が事実上なくなって、GoogleMap クラスから Marker オブジェクトを生成するようになりました。

マーカーのグループ管理ができなくなってちょっと不便に感じます。(v1 では、ローソンとファミマのマーカーを管理するのに、lawsonOverlay と famimaOverlay を用意しておけば良かった)

v2 のメリットは、マーカーがドラッグをサポートするようになったのと、情報ウィンドウ(ふきだし)が標準搭載されたことです。

#### 参考

* MarkerDemoActivity.java

### 図形(ラインとかポリゴンとか)描画
![Polygons](http://cl.ly/image/0c3i2E1z3V1r/gmap2_polygons.png)
これも大きく作り替えないといけないところです。

v1 では、Overlay の [draw](https://developers.google.com/maps/documentation/android/v1/reference/com/google/android/maps/Overlay.html#draw(android.graphics.Canvas, com.google.android.maps.MapView, boolean) を override して、Canvas の drawLine などの描画メソッドを呼び出す感じでした。

v2 では、Overlay がありませんし、Canvas を直接さわれる口がありません。
ではどうするかというと、[Polygon](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/Polygon) , [Polyline](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/Polyline) というクラスが用意されています。

GoogleMap クラスの [addPolygon](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/GoogleMap.html#addPolygon(com.google.android.gms.maps.model.PolygonOptions) , [addPolyline](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/GoogleMap.html#addPolyline(com.google.android.gms.maps.model.PolylineOptions) メソッドで追加します。Marker と同じく Overlay の概念はありません。

また add した Polygon や Polyline は、[remove](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/Polygon.html#remove() するか [GoogleMap.clear](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/GoogleMap.html#clear() するまで地図上に登録されています。

v2 になって、描画処理は重くなったなあという印象です。
v1 の感覚で図形を add しまくると、すごくもっさりします。画面に表示すべきものだけ、さらに非同期でパラパラと図形が描画されていくような処理を実装しないとストレスフルな感じになりそうです。

#### "表示している領域" の取得方法
v1 では、getMapCenter , getLatitudeSpan, getLongitudeSpan でなんとなく取得できました。
v2 では、GoogleMap.getProjection で得られる [Projection](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/Projection) クラスの getVisibleRegion から取得できます。

注意点は、回転している時とチルトしている時、いずれも "表示している範囲を包括する矩形" が得られるようですが、特にチルトしている時は奥行き分が領域に含まれますので、視点を倒せば倒すほど遠くが見える(領域が広がる)ことになります。画面に表示しているものだけ図形を描画する時など、奥行きはある程度のしきい値を持たないと範囲が広くなりすぎです。

#### ハマった所
Polygon は当然塗りつぶしができるわけですが、これポリゴンの座標群が「時計回り」でないと塗りつぶされません。地理情報システムの業界では、"ポリゴンの外周は時計回り、穴は反時計回り" という常識があるのですが、それに習ったものと思われます。この原因にたどり着くまで、始点と終点が一致してないのかな、少数の切り捨てで一致してないのかとか、小一時間悩みました。

#### 参考

* PolygonDemoActivity.java
* PolylineDemoActivity.java

### MapView とは一体？
冒頭で紹介した [com.google.android.gms.maps.MapView](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/MapView) ですが、地図の表示はできたものの、その後、位置を移動しようと

    GoogleMap gMap = ((MapView) findViewById(R.id.map)).getMap();
    gMap.animateCamera(CameraUpdateFactory.newLatLng(new LatLng(35, 130)));

というコードを実行した所、 ```java.lang.NullPointerException: CameraUpdateFactory is not initialized``` が発生してしまいました。え？どうやって移動するの？

#### 参考

* RawMapViewDemoActivity.java

## 増えた機能
v1 感覚だと使うのに苦労しそうな v2 ですが、冒頭で紹介した機能以外に追加されてる機能もあります。

### 情報ウィンドウ(ふきだし)
これこれ、マーカーをタップすると表示される「ふきだし」。
v1 には無かったので、自作か外部ライブラリ使いしか無かったんですよね。
サンプル MarkerDemoActivity.java で使い方が紹介されてます。

## タイルオーバーレイとグラウンドオーバーレイ
GIS'er 感涙の両機能。
タイルオーバーレイは、他のタイル地図サービスをオーバーレイできる、ってことですね。OpenStreetMap とか。iOS の [route-me](http://qiita.com/items/8d89eeea614ce4293514) みたいなことができると期待してます。だれかハックを！
![TileOverlay](http://f.cl.ly/items/470S1b1B3a1F3F3z093i/gmap2_tileoverlay.png)

グラウンドオーバーレイは、画像ファイルに位置情報を与えてやると、それが地面に張り付いたように表示されるというものです。
![GroundOverlay](http://f.cl.ly/items/470s2M3g0w3Z2H423B3t/gmap2_groudoverlay.png)

タイルオーバーレイのサンプルは TileOverlayDemoActivity.java 、グラウンドオーバーレイのサンプルは GroundOverlayDemoActivity.java です。

## 雑感
Google Map Android API v2 をざっと使ってみての感想です。

v1 と互換性がなくなり、使い方がガラっと変わったのはある程度想定していました。2次元と3次元の溝は深いです。

Android と関係ないですが、iOS の MapKit も同じようなことになるのではないかと思っています。(地図アプリは 3D に対応しましたが MapKit は 2D のまま。3D用のMapKit は "まったく別のもの" として提供されるのではないかと。)

導入の仕方が難しくなっちゃったなー、と。
なぜ Google API Console で管理するのはよいとしても、ライブラリプロジェクトを参照しないといけないのか、とか。
(実は Fragment 初めて使いました…)

あと、Google Play Service や API Console との結びつきが強くなり、利用状況がモニタリングできる環境が整いつつあります。こりゃ課金も現実味を帯びてきたなあ、と。

そんなわけで、導入がちょいと面倒だったり、v1 と互換性まったくないですけど、なにしろ Googleマップアプリのあの地図レンダリングと機能がアプリに組み込めるのは嬉しい限りです。新たに地図アプリ作るなら使わない手はないですね！

まだ使いこなしてはいないので、順次 Post していきたいと思います。
最後に拙作の Android アプリ「HexRinger」の Map API v2 対応試作版をステマして終わります。

![HexRinger](http://cl.ly/image/1n0f1e0c1r3C/gmap2_hexringer.png)
