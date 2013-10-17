---
layout: post
title: "Googleマップ禁止令が出たのなら、電子国土地図をGoogleマップで使えばいいじゃない"
date: 2013-10-11 13:55
comments: true
categories: [Geo, GoogleMapsAPI, JavaScript, HTML]
---
「領土等が正しく表示されていない」として、国や自治体、国公立の機関に、「Google マップ禁止令」が出ているそうです。
<!--more-->

----
## 2013.10.17 追記

[コメント](http://qiita.com/amay077/items/979dfc858a21c8bbb7a9#comment-6d0be86c2ba90df50808)で頂いていますが、現在は、 **「日本向けにローカライズされた」** Googleマップ側でも表記が修正されたようで、当初「問題がある」と言われていた地域は問題が解消されているように見えます。
**ただし、下記のように Google Maps API で「region=JP」が指定されている場合に限ります。**

* [Google Maps API v3 で 日本海（東海）と「東海」が併記されるのを避けるには - わからん](http://d.hatena.ne.jp/kitokitoki/20130415/p1)

これによって「Google禁止令はもはや無用である」と考えることもできますし、「依然として国の制御下には無い」と考えることもできます。

以下の情報は必要なくなるかも知れませんが、今後「Googleマップ以外の選択肢」の一つとして参考になれば幸いです。

-- 追記ここまで --

----

* [グーグルマップの「利用禁止令」 竹島や北方領土が「日本名でない表記」 政府が自治体などに要請](http://www.huffingtonpost.jp/2013/09/29/google-map-forbidden_n_4011180.html)
* [Googleマップ禁止令?! 制作会社の苦悩と対応を考える。 - NAVER まとめ](http://matome.naver.jp/odai/2138082171982287301)
* [大学公式ページでのGoogleMap禁止令と彩雲 : ５号館のつぶやき](http://shinka3.exblog.jp/20690859/)
* [Twitter - うちの大学でも例の「グーグルマップ禁止令」によるサイトのチェックが始まった…](https://twitter.com/nissyyu/status/387870282570272768)

これ自体の是非はいろいろあるようですが、公的機関の情報表示が、国によって「under control」じゃないのはイカン、というのはまあ分かりますよ。

で、対策としてはいくつかあります。。。

#### 画像の地図や PDF に変える

* どんな時代錯誤ですか

#### 他の地図APIを使う

* Yahoo Japan、Mapion、Bing、OSM とか。
* Google Map API からこれらに変更する手間は結構かかるんではないかと。特に地図サイトを作りこんでいるならなおさら。
* Yahoo Japan、Mapion は、規約的に公的機関で無償で使用OK でしたっけ？
* Bing, OSM は「制御下には無い」でしょう。

#### GoogleMapの問題のある表記を「正しい表記」に上書きする

* [Googleマップ禁止令?! 制作会社の苦悩と対応を考える。](http://matome.naver.jp/odai/2138082171982287301) で実践されていた手法、ある意味驚きました（^_^;）
* これ→[日本政府に「禁止されない」 Google Maps - jsdo.it - Share JavaScript, HTML5 and CSS](http://jsdo.it/toaSoku/gmapforjapan)
* スクロール時にチラッと元表記が見えちゃうのと、元地図画像の変化に追従するのが難しいんですよね。

いずれも対応コストや規約、ユーザビリティを考えると決め手に欠けます。

## ではどうするか？

[電子国土](http://portal.cyberjapan.jp/)という、国土交通省国土地理院が作って提供している地図データ、および地図システムがあります。

身内ならこれ使えよ、という話です。実際、大学などに送られた通知には、こちらを利用するための「相談窓口」が記載されているようです。

ただし、Google Map API から電子国土の地図APIに移行することは、前述の通それなりのコストが発生します。

ので、ここでは、 **「Google Map API を使って電子国土地図を利用する方法」** を紹介します。

Google Map API には他の地図データを Overlay（重ねあわせ）する機能が備わっており、これを利用します。

既に実現されてる方々がいらっしゃいます。

* [Googleマップを使って国土地理院の地図を見る](http://user.numazu-ct.ac.jp/~tsato/webmap/map/gmap2.html?data=djws)
* [y2blog » Google Maps APIを用いて電子国土V4背景地図を表示する](http://y2web.net/blog/computer/webmap/show_cj4_tiles_on_google_map_system-3355/)

これらのサイトを参考にしてもよいですが、ここでは、Googleマップで電子国土地図を使う、 最もシンプルな実装例を紹介します。

### 1. Googleマップを表示するページを用意する

ここでは、仮の「運用中のサイト」として、Google マップを使うシンプルなページを用意しました。

* [Google Maps Javascript API v3 の使用例](http://jsdo.it/amay077/tM0q)

HTML+Javascript のソースコードはこんな感じ([Geekなぺーじ:Google MAPS JavaScript APIの単純な例](http://www.geekpage.jp/web/google-maps-api/v3/helloworld.php) を参考にさせて頂きました)

```html google.html
<!DOCTYPE html>
<html>
  <head>
    <style type="text/css">
      html { height: 100% }
      body { height: 100%; margin: 0px; padding: 0px }
      #map { height: 100% }
    </style>
 
    <script src="http://maps.google.com/maps/api/js?v=3&sensor=false"
        type="text/javascript" charset="UTF-8"></script>
 
    <script type="text/javascript">
    function init() {
      var opts = {
        zoom: 5,
        mapTypeId: google.maps.MapTypeId.ROADMAP,
        center: new google.maps.LatLng(39, 135)
      };
 
      var map = new google.maps.Map(document.getElementById("map"), opts);
    }
    </script>
  </head>
 
  <body onload="init()">
    <div id="map"></div>
  </body>
 
</html>
```

### 2. 電子国土地図を表示するように置き換える

* [Google Maps Javascript API v3 での電子国土地図の表示例](http://jsdo.it/amay077/wFtJ)

![](https://dl.dropboxusercontent.com/u/264530/qiita/denshikokudo_on_googlemap_01.png)

表示が電子国土の地図に置き換わっているのが確認できると思います。右上の地図タイプ切り替えは要らないので消してます。あと、「電子国土」のロゴの表示が必要なので、左下に置いています。

という処理を追加したのが、下のコード。
変更が１行、追加行が６０行くらい（ちと長い?）です。
変更箇所は「←」で、追加箇所は「↓↓」「↑↑」で示しています。
[gist](https://gist.github.com/amay077/6928205/revisions) でも diff を見られます。

```html denshikokudo.html
<!DOCTYPE html>
<html>
  <head>
    <style type="text/css">
      html { height: 100% }
      body { height: 100%; margin: 0px; padding: 0px }
      #map { height: 100% }
    </style>
 
    <script src="http://maps.google.com/maps/api/js?v=3&sensor=false"
        type="text/javascript" charset="UTF-8"></script>
 
    <script type="text/javascript">
    function init() {
      var opts = {
        zoom: 5,
        mapTypeId: "DenshiKokudo",   // ← 電子国土を使うように変更
        center: new google.maps.LatLng(39, 135)
      };
 
      var map = new google.maps.Map(document.getElementById("map"), opts);
 
      // ↓↓電子国土地図を Overlay する↓↓
      map.setOptions({
        mapTypeControl: false // 右上の地図タイプ選択を消す
      });
 
      // via http://portal.cyberjapan.jp/portalsite/version/v4/index.html
      var dataset = [null, null, null, null, null, "JAIS","JAIS","JAIS","JAIS", "BAFD1000K",
        "BAFD1000K", "BAFD1000K", "BAFD200K","BAFD200K","BAFD200K", "DJBMM","DJBMM",
        "DJBMM", "FGD"];
 
      map.mapTypes.set("DenshiKokudo", {
        name:"電子国土",
        tileSize:new google.maps.Size(256,256),
        minZoom:5,
        maxZoom:18, 
        getTile:function(tileCoord, zoom, ownerDocument) {
          var img = ownerDocument.createElement("img");
          img.style.width = "256px";
          img.style.height = "256px";
          var x = (tileCoord.x % Math.pow(2, zoom)).toString();
          var y = tileCoord.y.toString();
          var tileX = "0000000".substr(0, (7 - x.length)) + x;
          var tileY = "0000000".substr(0, (7 - y.length)) + y;
          var dir = "";
          for( var i = 0; i < 6; i++ ) {
            dir += tileX.charAt(i) + tileY.charAt(i) + "/";
          }
          img.src = "http://cyberjapandata.gsi.go.jp/sqras/all/" + dataset[zoom] + "/latest/" + zoom + "/" + dir + tileX + tileY + ".png";
          return img;
        }
      });
 
      // 左下に電子国土ロゴを表示する
      var logo = document.createElement('DIV');
      logo.style.padding = '3px';
      map.controls[google.maps.ControlPosition.LEFT_BOTTOM].push(logo);
      logo.innerHTML = '<a href="http://portal.cyberjapan.jp/portalsite/kiyaku/index.html" target="_blank"><img style="width:32px, height:32px" src="http://cyberjapan.jp/images/icon01.gif" alt="電子国土" /></a>';    
      // ↑↑電子国土地図を Overlay する↑↑
    }
    </script>
  </head>
 
  <body onload="init()">
    <div id="map"></div>
  </body>
 
</html>
```

### 不安とか懸念とか…

#### 地図の表示が Google に比べて…

まあ、そうですね。国土地理院にどんどんフィードバックしていきましょう。

#### 「試験公開」ってなってるんですけど…

なにが「試験公開」なのかよくわからないです。電子国土Web.NEXT が試験公開で、地図画像自体は「正式」公開なのでしょうか。。。

運用予定についてのページ [電子国土Webシステムの今後の予定について](http://portal.cyberjapan.jp/portalsite/version/djapan_yotei.html#v234) をチェックしておくと良さそうです。

#### 利用規約とかどうなってるの？

[電子国土Webシステムの利用に関するフロー](http://portal.cyberjapan.jp/portalsite/q_and_a/index.html) を見てください。

## まとめ

いかがでしょうか？

現在の Google マップを使ったページに、２箇所の修正を加えるだけで、「問題のない」電子国土の地図に切り替える事ができます。これは今回の問題の一つの解決方法になるのではないでしょうか？

個人的には、今回の「Googleマップ禁止令」は、国が所有する膨大な地図データを広く使ってもらえるチャンスだと思っています。

電子国土地図の表示例を見てもらえれば分かりますが、地図としての見た目はともかく、データの量・精度については、Googleマップ（というかゼンリン）と肩を並べる（あるいは上回るところもある）と思っています。

せっかく税金で作られている地図なんですから、上手に活用していけば日本全体の利益になるんじゃないかと思います。

最後にお約束で、本件のご利用は自己責任でお願いします。