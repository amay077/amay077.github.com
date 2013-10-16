---
layout: post
title: "総務省のデータを Elasticsearch にぶち込んで、緯度経度から市区町村の何丁目までを取り出す"
date: 2013-09-10 21:13
comments: true
categories: [Elasticsearch, Geo]
---
いわゆる「逆ジオコーディング」と呼ばれる機能ですが、きっかけはこれら２つの記事です。
<!--more-->
* [PHP - 総務省のデータを使って、緯度経度から市区町村の何丁目までを取り出す](http://qiita.com/hamichamp/items/ac9e80f1078febb9f1b9)
* [PostgreSQL - 国土交通省のデータを使って、緯度経度から市区町村までを取り出す](http://qiita.com/masuidrive/items/21a282c7bf54fd6a4985)

Solr や Elasticsearch でも同じことができるのでは、という事で Elasticsearch でやってみました。

## Elasticsearch の導入

は、

* [Elasticsearch で位置情報を検索する手順 - Experiments Never Fail](http://amay077.github.io/blog/2013/09/09/using-spatialdata-with-elasticsearch/)

をご覧ください。

## 1. 総務省のデータをダウンロードする

* [PHP - 総務省のデータを使って、緯度経度から市区町村の何丁目までを取り出す](http://qiita.com/hamichamp/items/ac9e80f1078febb9f1b9)

とほぼ同じですが、「世界測地系緯度経度・G-XML形式」ではなく、 **「世界測地系緯度経度・shape形式」** を使います。

一応再掲すると、

1. [http://e-stat.go.jp/SG2/eStatGIS/page/download.html](http://e-stat.go.jp/SG2/eStatGIS/page/download.html) へ行く。
2. 左から「平成22年度国勢調査（小地域）」を選ぶ。
3. 「男女別人口総数及び世帯総数」（←なんでもいい）を選択して、「統計表各種データダウンロードへ」を押す。
4. 「世界測地系緯度経度・Shape形式」のデータ（下図参照）をダウンロードする。ダウンロードしたら zip ファイルのまま置いといて。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/implementing_reverse_geocoding_using_elasticsearch_01.png)

## 2. Shapefile を GeoJSON 形式に変換する

Elasticsearch へ投入できるデータ形式は JSON なので、ダウンロードした Shape形式のデータを JSON 形式の地理空間拡張である GeoJSON 形式に変換します。

スクリプトを書いてもできますが面倒なので、便利なオンラインツールに頼ることにします。

* [Free on-line GIS data format and coordinates converter](http://converter.mygeodata.eu/#convertVector)

手順は、

1. 上記サイトへ行き、「Run vector converter」を押す。
2. 「ファイルを選択」で、さっきダウンロードした ZIP ファイルを指定し、「Send ZIP File」を押す。
3. "Datasets description" というページになったら、その最下部にある「Chack available operations」を押す。(ここでは、不要なデータ項目を除外できるのだけど、面倒なので割愛)
4.  ”Export to format:” で "GeoJSON" を選択。それ以外はそのままで「Proceed selected operation」を押す。
5. しばらく待つと、「Download the ZIP file」リンクが表れるので、押して変換結果をダウンロードする。

ダウンロードした ZIP ファイルを解凍すると、「xxx.json」ファイルが見つかります。それをテキストエディタで開くと、``features`` 以下に、住所エリアの情報が1行ずつ出力されています。

試しに1行取り出して、JSON を整形（見やすいよう適宜省略）してみると次のようになります。

```javascript town.json
{
    "type": "Feature",
    "properties": {
        "KEN_NAME": "愛知県", 
        "GST_NAME": "名古屋市", 
        "CSS_NAME": "中区", 
        "MOJI": "本丸", 
        …省略…
    }, 
    "geometry": {
        "type": "Polygon",
        "coordinates": [
            [[136.895888, 35.187236], 
             [136.897375, 35.187357], 
              …中略… 
             [136.895888, 35.187236]]
        ]
    }
}
```

これは「愛知県名古屋市中区本丸」のデータですね。
``properties`` 以下は、この住所エリアの属性情報を示しています。
``geometry`` 以下が、この住所エリアの位置情報（ポリゴン）を示しています。

## 3. Elasticsearch にスキーマを定義する

Elasticseach はスキーマフリーですが、位置情報のところだけは明示的に宣言しないといけないらしいので、下のようなコマンドを実行して定義します。

あ、ここでは、

* Index名 : towns
* Type名 : town

としています。

「Index や Type って何？」という方は

* [Elasticsearch用語の適当なまとめ](http://qiita.com/ise_daisuke/items/5e10e0b3ef9dffed08a9)

をどうぞ。

ではコマンドです。

```sh
curl -XPUT 'http://localhost:9200/towns/'

curl -XPUT 'http://localhost:9200/towns/town/_mapping' -d '
{
    "town" : {
        "properties": {
            "geometry": {
                "type": "geo_shape", 
                "tree": "quadtree",
                "precision": "1m"
            }
        }
    }
}'
```

``properties.geometry`` は、「geo_shape」 として扱う事を宣言しています。他の２つの設定は、インデックスの種類と精度を意味しますが、よく分かってません。

* [Geo Shape Type | Reference Guide | Elasticsearch](http://www.elasticsearch.org/guide/reference/mapping/geo-shape-type/)

で勉強しましょう。

## 4. データを Elasticsearch に投入する

さて、いよいよこの「1行1JSON」のデータを、1行ずつ、Elasticsearch に投入します。
先の xxxx.json を置換なり何なりを駆使して、スクリプトにしちゃうのがてっとり早いでしょう。(json ファイルは Shift-jis なので、UTF-8 に変換しておきましょう。）

1行のデータを投入するコマンドは次のようになります。

```
curl -XPUT 'http://localhost:9200/towns/town/1' -d '{
    "type": "Feature",
    "properties": {
        "KEN_NAME": "愛知県", 
        "GST_NAME": "名古屋市", 
        "CSS_NAME": "中区", 
        "MOJI": "本丸", 
        …省略…
    }, 
    "geometry": {
        "type": "Polygon",
        "coordinates": [
            [[136.895888, 35.187236], 
             [136.897375, 35.187357], 
              …中略… 
             [136.895888, 35.187236]]
        ]
    }
}'
```

``towns/town/1`` の最後の「1」のところは、連番にする必要があります。(オートインクリメントとかないのかな？)

全件を PUT すつスクリプトファイルは、下の画像のような感じになると思います。(データのライセンスがどうか分からないのでスクリプトファイル自体を公開するのはやめておきます)

![img](https://dl.dropboxusercontent.com/u/264530/qiita/implementing_reverse_geocoding_using_elasticsearch_02.png)

これをTerminal で実行すると、数分かからずに Elasticsearch にデータが投入完了します。

``curl -XGET 'http://localhost:9200/towns/town/1'`` などを実行すれば、正しくデータが登録できたか確認できます。

## 5. 検索してみる

ついに来ました。
では、緯度経度を与えて住所が返ってくるところ、やってみましょう。

Elasticsearch では検索クエリも JSON で書きます。

例えば、[大須観音駅](http://yahoo.jp/zsXYL2) らへんの緯度経度(35.1613077,136.898282)の住所で検索する場合は、次のようにします。

```
curl -XPOST 'http://localhost:9200/towns/town/_search' -d '{
    "query": {
        "filtered" : {
            "query" : {
                "match_all" : {}
            },
            "filter" : {
                 "geo_shape": {
                    "town.geometry": {
                        "shape": {
                            "type" : "envelope",
                            "coordinates" : [[136.898282, 35.1613077], [136.898282, 35.1613077]]
                        }
                    }
                }
            }
        }
    }
}'
```

``geo_shape`` フィルタを使い、条件に envelople(左上〜右下の領域) を指定します。今は「点」での検索をしたいので、左上、右下に同じ座標を指定します。
注意点は、 **「経度, 緯度」** の順であることです。

※ [``geo_distance``](http://www.elasticsearch.org/guide/reference/query-dsl/geo-distance-filter/) というフィルタもありますが、こちらはデータが「点」専用なようで、今回のような「ポリゴン」には使えませんでした。

さて、上のコマンドを実行すると、下のような結果が得られます。(整形、省略済)

```
{
  "took":1,
  "timed_out":false,
  "_shards":{
    "total":5,
    "successful":5,
    "failed":0
  },
  "hits":{
    "total":1,
    "max_score":1.0,
    "hits":[{
      "_index":"towns",
      "_type":"town",
      "_id":"29",
      "_score":1.0, 
      "_source" : { 
        "type": "Feature", 
        "properties": { 
          "KEN_NAME": "愛知県", 
          "GST_NAME": "名古屋市", 
          "CSS_NAME": "中区", 
          "MOJI": "大須２丁目", 
          …以下省略…
```

はい、「緯度経度(35.1613077,136.898282)」の住所は「愛知県名古屋市中区大須２丁目」であることが取得できました。

## まとめ

いかがでしょうか、Elasticsearch でも逆ジオコーディングの実装が、簡単にできることが分かりました。

PostGIS、MongoDB との対比では、

* セットアップが簡単
* REST API なので直接他のサービスとマッシュアップできる

あたりがメリットでしょうか。

逆にデータの取り込みはひと工夫必要で、PostGIS の方が簡単です。
このデータ用の River plugin を作ればよいのでしょうが、方法がさっぱり…。

最後に、

[総務省のデータを使って、緯度経度から市区町村の何丁目までを取り出す](http://qiita.com/hamichamp/items/ac9e80f1078febb9f1b9)

> 実は、データをダウンロードするのが、一番手間がかかります・・・。

でも言われていますが、まったくその通りです。
１市区町村ずつダウンロードとか正直やってられません。

総務省でも国土交通省でもどちらでも良いのですが、
**「全国の大字境界＋属性データを一括入手する方法」** を用意して欲しいものです。