---
layout: post
title: "地理院地図の標高タイル(CSV)を描画してみた"
date: 2013-11-05 21:11
comments: true
categories: [Geo, GoogleMapsAPI, HTML5, JavaScript]
---
国土地理院が提供している API の一つに「標高タイル」というものがあります。
<!--more-->
タイルというと、Googleマップや OpenStreetMap などの Web地図では通常、画像を指しますが、標高タイルAPIでは **「画素毎の高度(ｍ)」** が取得できます。

* [標高タイル仕様 - 地理院地図](http://portal.cyberjapan.jp/help/development/demtile.html)
* [サンプルURL](http://cyberjapandata.gsi.go.jp/xyz/dem/14/14547/6463.txt)

これは面白い、ということで使ってみました。

## サンプル

* [地理院地図の標高タイル(CSV)を描画してみた - jsdo.it](http://jsdo.it/amay077/jjod)

### Google Map 

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/using_gsimap_dem_csv_api_01.png)

### 標高タイルAPI で取得した標高値を描画

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/using_gsimap_dem_csv_api_02.png)

## 何をしているか？

下は、このサンプルのコードの抜粋ですが、ポイントは２つ

* getTile で通常 img 要素を生成して返すが、代わりに canvas 要素に返す
* 標高API をコールして得られた CSV をパースし、 高さに応じた色を計算して、canvas に矩形を描画する

さすがに1ピクセル毎に描画すると重すぎるので、初期値では 16ピクセルずつに間引きしています(画面の DotSize で変更できます)。

クライアント側でレンダリングしているので、色などが動的に変更できます。

```javascript 
    map.mapTypes.set("GsiMaps", {
      name:"標高タイル",
      tileSize:new google.maps.Size(256,256),
      minZoom:14, // 標高タイルは Lv:14 しか用意されてないので
      maxZoom:14, 
      getTile:function(tileCoord, zoom, ownerDocument) {
          
        // 普通は img だけど、標高タイルは CSV で画素毎の標高値が取得できるので、
        // クライアント側で描画するために Canvas を使う
        var canvas = ownerDocument.createElement("canvas");
        canvas.width = 256;
        canvas.height = 256;
        
        var x = (tileCoord.x % Math.pow(2, zoom)).toString();
        var y = tileCoord.y.toString();
        
        // 各画素の標高値を取得する
        canvas.tileUrl = "http://cyberjapandata.gsi.go.jp/xyz/dem/" + zoom +  "/" + x + "/" + y + ".txt";
        // 標高を描画する
        renderDem(canvas);
        renderedTiles[canvas.tileUrl] = canvas; // タイル再描画の為にとっておく
        return canvas;
      }
    });
    
    // 標高タイルを描画する
    function renderDem(canvas) {
      var ctx = canvas.getContext('2d');
      ctx.clearRect(0, 0, 256, 256);
      
      $.get(canvas.tileUrl, function(data) {
        // CSV が得られるのでパース
        var lines = data.split(/\r\n|\r|\n/);
        for (var i = 0; i < lines.length; i+=dotSize) {
          var cols = lines[i].split(',');
          for (var j = 0; j < cols.length; j+=dotSize) {
            if (cols[j] == 'e') { // エラーの画素には 'e' が入ってる
              continue;
            }
              
            // 標高0ｍ を startColor、標高1000ｍを endColor としたグラデーション色を設定する。
            ctx.fillStyle = $.xcolor
            .gradientlevel(startColor, endColor, cols[j] / 1000.0 * 100.0, 100)
            .getCSS();
            ctx.fillRect(j, i, dotSize, dotSize);
          }
        }
      });
    }
```

## まとめ

ただの標高値を地図に表すなら画像でいいじゃん！とか言われそうですが、よいアイデアが浮かばなかったのでまずは素直に使ってみました。

たとえば、移動手段による移動コストの違い(車だと坂道余裕だけど自転車だとキツい)みたいなのを視覚化するのに使えるような気がします。

今回は、Canvas を使ったのでこの程度ですが、WebGL とかを使えば、Google Map の地形図に負けない、美しい3D地図が描画できるはずです。

他に例を見ない、野心的な試みだと思うので、何か面白い使い方ができるといいなと思います。
