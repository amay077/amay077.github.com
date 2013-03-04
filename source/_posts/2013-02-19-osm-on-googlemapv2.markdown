---
layout: post
title: "Google Maps Android API v2 で OpenStreetMap を表示する"
date: 2012-12-26 00:18
comments: true
categories: [Android, GoogleMapsAPI, OpenStreetMap]
---

この記事は [FOSS4G Advent Calendar 2012](http://atnd.org/events/34052) の 12/26 の記事です。

ベクトル地図が扱える新しい Google Maps Android API v2 については、[Google Map Android API v2 の v1 からの変更点メモ](http://qiita.com/items/7ad0244c0fb4b431e090) で書きました。

ここでは、v2 で新しく追加された ``TileOverlay`` を使って、OpenStreetMap を重ねてみます。

<!-- more -->

## UrlTileProvider を使って OpenStreetMap を表示する
SDK に同梱されるサンプル /extras/google/google_play_services/samples/maps の TileOverlayDemoActivity.java を見れば一目瞭然なので、それをベースにします。

### サンプルのコード
``` java TileOverlayDemoActivity.java
/** This returns moon tiles. */
private static final String MOON_MAP_URL_FORMAT =
        "http://mw1.google.com/mw-planetary/lunar/lunarmaps_v1/clem_bw/%d/%d/%d.jpg";

private GoogleMap mMap;

private void setUpMap() {
    mMap.setMapType(GoogleMap.MAP_TYPE_NONE);

    TileProvider tileProvider = new UrlTileProvider(256, 256) {
        @Override
        public synchronized URL getTileUrl(int x, int y, int zoom) {
            // The moon tile coordinate system is reversed.  This is not normal.
            int reversedY = (1 << zoom) - y - 1;
           String s = String.format(Locale.US, MOON_MAP_URL_FORMAT, zoom, x, reversedY);
            URL url = null;
            try {
                url = new URL(s);
            } catch (MalformedURLException e) {
                throw new AssertionError(e);
            }
            return url;
        }
    };
    mMap.addTileOverlay(new TileOverlayOptions().tileProvider(tileProvider));
}
```
修正前のコードは、Google Moon のタイル画像を使用しています。

これを OpenStreetMap を使用するように改造します。

``` java OsmTileOverlayDemoActivity.java
/** This returns moon tiles. */
private static final String OSM_MAP_URL_FORMAT =
        "http://tile.openstreetmap.org/%d/%d/%d.png";

private GoogleMap mMap;

private void setUpMap() {
    mMap.setMapType(GoogleMap.MAP_TYPE_NONE);

    TileProvider tileProvider = new UrlTileProvider(256, 256) {
        @Override
        public synchronized URL getTileUrl(int x, int y, int zoom) {
            String s = String.format(Locale.US, OSM_MAP_URL_FORMAT, zoom, x, y);
            URL url = null;
            try {
                url = new URL(s);
            } catch (MalformedURLException e) {
                throw new AssertionError(e);
            }
            return url;
        }
    };
    mMap.addTileOverlay(new TileOverlayOptions().tileProvider(tileProvider));
}
```

できました。うーん、簡単すぎる。
URL は OpenStreetMap のものを使います。y軸の値は、Google Moon では逆順となっていたのを正順のまま使用するだけです。

こんな感じで表示できます。

![OpenStreetMap on Google Map API](https://dl.dropbox.com/u/264530/qiita/advent2012_osm.png)

移動、拡大・縮小だけでなく、API v2 の恩恵で、回転やチルトもできるのが嬉しいですね。

## TileOverlay を透過させる
さて、ベース地図を Google から他のものに差し替えてしまうならこれまでの使い方で十分でしょう。しかし Google のベクトル3Dグリグリ地図をベース地図として使いたいとは誰しもが思うことでしょう。

ここでは、Google地図の上に TileOverlay を透過で表示することにチャレンジしてみます。
ケースとしては、雨雲レーダーのメッシュや、統計メッシュなどを重ね合わせる事が考えられます。

さて、API v2 のもう一つの新機能 GroundOverlay には [setTransparentcy](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/GroundOverlay#setTransparency(float) というズバリなメソッドがあり、それを使えば一発です。

しかし、TileOverlay とその関連クラスには、透過に関するメソッドは見当たりません。
そこで TileProvider でダウンロードされた画像データを直接弄って、透過にします。

TileProvider は文字通り [Tile](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/Tile) を Provide します。そしてこの Tile はタイル画像データそのものです。

[Tile.data](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/Tile#data) の説明には次のように記述があります。

> A byte array containing the image data. The image will be created from this data by calling [decodeByteArray(byte[], int, int)](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/null#decodeByteArray(byte[], int, int)).

つまりこのプロパティの中身を透過させてあげれば良さげ、という事になります。

上記のコードで使用した UrlTileProvider の [getTile](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/UrlTileProvider#getTile(int, int, int) を override して…と思ったら、
＿人人人人人人人人人＿
＞　突然の final！　＜
￣^Y^Y^Y^Y^Y^Y^Y^￣
という事で override できません。

仕方ががないので、独自の TileProvider を別途用意して、UrlTileProvider を内包する形で ``TransparencyUrlTileProvider`` というクラスを実装します。


```java TransparencyUrlTileProvider.java
public class TransparencyUrlTileProvider implements TileProvider {
	private static final String OSM_MAP_URL_FORMAT = "http://tile.openstreetmap.org/%d/%d/%d.png";

	private int _transparency; // 透過率(0〜255)
	private UrlTileProvider _osmTileProv; // 内包する TileProvider
	
	public TransparencyUrlTileProvider(int width, int height, int transparency) {
		_transparency = transparency;
		
		_osmTileProv = new UrlTileProvider(width, height) {
			@Override
			public URL getTileUrl(int x, int y, int zoom) {
	            String s = String.format(Locale.US, OSM_MAP_URL_FORMAT, zoom, x, y);
	            URL url = null;
	            try {
	                url = new URL(s);
	            } catch (MalformedURLException e) {
	                throw new AssertionError(e);
	            }
	            return url;
			}
		};
	}
	
	@Override
	public Tile getTile(int x, int y, int zoom) {
		Tile tile = _osmTileProv.getTile(x, y, zoom);
		
		// TODO ここで Tile の透過処理を行う
		
		return tile;
	}

}
```

使う側は、こんな感じになります。

``` java OsmTileOverlayDemoActivity.java
private GoogleMap mMap;

private void setUpMap() {
    // mMap.setMapType(GoogleMap.MAP_TYPE_NONE); ベース地図は消さない

    mMap.addTileOverlay(
        new TileOverlayOptions()
        .tileProvider(
            new TransparencyUrlTileProvider(256, 256, 100)));
}
```

ここまでで改造前と同じく OpenStreetMap が「非透過で」表示されるのは確認できます。

次にいよいよ Bitmap の透過処理です。
まず、Tile から Bitmap を抜き出します。API リファレンスによると、[Tile.data](https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/Tile#data) というメンバがあるハズが…見つかりません。代わりに ``Tile.bM`` という byte[] なメンバがあります。こいつで間違いないでしょう。

Tile.bM の byte[] から Bitmap インスタンスを生成します。

    Bitmap bitmap = BitmapFactory.decodeByteArray(tile.bM, 0, tile.bM.length);

次に透過処理ですが、Android ではちょっと面倒なようです。
以下のサイトを参考にさせて頂いて、関数を作成しました。

* [Android: Bitmap の背景を透明にする - 入隠者通信 ～病を嗜む～](http://d.hatena.ne.jp/hypercrab/20110730/1312038162)

```java makeTransparentBmp.java
private static Bitmap makeTransparentBmp(final Bitmap bmp, int transparency) {
     int width = bmp.getWidth(); 
     int height = bmp.getHeight(); 
     int[] pixels = new int[width * height]; 

     Bitmap bitmap = Bitmap.createBitmap(width,height,Bitmap.Config.ARGB_8888 );
     bmp.getPixels(pixels, 0, width, 0, 0, width, height); 
     for (int y = 0; y < height; y++) { 
       for (int x = 0; x < width; x++) { 
      	 int pixel = pixels[x + y * width];
      	 pixels[x + y * width] = Color.argb(transparency, 
      			 Color.red(pixel), Color.green(pixel), Color.blue(pixel)); 
       } 
     } 
     bitmap.eraseColor(Color.argb(0, 0, 0, 0)); 
     bitmap.setPixels(pixels, 0, width, 0, 0, width, height); 
     
     return bitmap;
}
```

では TODO の所に組み込みます。

```java TransparencyUrlTileProvider.java
public class TransparencyUrlTileProvider implements TileProvider {

    <前略>
	
	@Override
	public Tile getTile(int x, int y, int zoom) {
		Tile tile = _osmTileProv.getTile(x, y, zoom);
		
		// Tile の透過処理を行う
       Bitmap bmp = BitmapFactory.decodeByteArray(tile.bM, 0, tile.bM.length);
       Bitmap transparentBmp = makeTransparentBmp(bmp, _transparency);

       // Tile を作り直す
		ByteArrayOutputStream bos = new ByteArrayOutputStream();
		transparentBmp.compress(CompressFormat.PNG, 100, bos);
		Tile tranparentTile = new Tile(tile.width, tile.height, bos.toByteArray());
		
		return tranparentTile;
	}

	<以下略>
}
```

動かしてみます。

![OpenStreetMap with Google Map API](https://dl.dropbox.com/u/264530/qiita/advent2012_osmwithg.png)

これは、GoogleMap の衛星写真の上に OpenStreetMap を透過して重ねた例です（分かりづらい

## まとめ
このように Google Maps Android API v2 では、TileProvider を使って、タイル地図画像を簡単に表示させることができます。

Google Maps Javascript API や、MapKit でも他のタイル地図画像を利用することはできましたが、それらよりもより簡単に使えます。ハックというよりも API が公式にサポートしている、という感じです。

これまで Android には、Javascript の OpenLayers や、 iOS の route-me のような、地図タイルデータソースを扱える地図SDKはありませんでした（いや OsmDroid くらいか）

それを Google Maps Android API v2 がサポートしたのですから使わない手はありません。地図SDK としては一番高性能で事実上標準なのですから。

これに、OpenStreetMap や電子国土地図、衛星画像などの背景地図や、統計データメッシュや、アメダスなどの主題図的なタイル地図が重ねられるといろいろできそうだなあ、という感じです。
(Yahoo! さんの[雨雲レーダー](http://weather.yahoo.co.jp/weather/zoomradar/) のタイル画像もこっそり試してみて「こりゃ面白い」と思ったので公式に提供して欲しいですｗ)

私にはタイル地図データを作る知識は無いので、タイル地図のポータルみたいなものがあるといいなあと思います。 [地図タイル工法協会](https://www.facebook.com/chitaikyo) さんよろしくおねがいします。

というわけで、Android で地図使いたいなら(今のところ) Google Maps API v2 一択！ 他社さんもガンバレ！

※あれ？このネタどこが FOSS4G だ？ま、いっか。