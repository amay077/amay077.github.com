---
layout: post
title: "Elasticsearch で位置情報を検索する手順"
date: 2013-09-09 21:08
comments: true
categories: [Elasticsearch, Geo]
---
Elasticsearch は、オープンソースの全文検索エンジンです。Apache Solr と並んでよく取り上げられるようになってきました。

位置情報の検索機能も標準搭載しているとのことで、試しに使ってみました。
<!--more-->
## Elasticsearch の導入

下の情報が大変参考になりました。(環境は Mac。事前に Java と homebrew の導入が必要です）

* [elasticsearch - ElasitcSearch ことはじめ - Qiita [キータ]](http://qiita.com/Konboi@github/items/56f0aaca77db5df027af)

## 試しに使ってみる

下のサイトが大変参考なりました。

* [Elasticsearch を試してみる - ようへいの日々精進](http://inokara.hateblo.jp/entry/2013/09/07/153826)

## 位置情報を検索してみる

下のようなスキーマのデータを登録して検索する想定です。

```javascript venue_example.json
{
    "name" : "Tokyo St",
    "pin" : {
        "location" : {
            "lat" : 35.68,
            "lon" : 139.76
        }
    }
}
```

### 1. スキーマを登録する

Elasticsearch は基本的にはスキーマレスで動くのですが、位置情報を表す項目は、明確にスキーマを定義する必要があるようです。

データ投入の前にそれを行います。

```sh 
curl -XPUT 'http://localhost:9200/myvenues/'

curl -XPUT 'http://localhost:9200/myvenues/venue/_mapping' -d '
{
    "venue" : {
        "properties" : {
            "pin" : { "type" : "geo_point" }
        }
    }
}'
```

ここでは、``venue`` のプロパティ群の内の ``pin`` 項目は、位置情報(geo_point)だよ、と定義しています。

### 2. データを投入する

2件ほど、テストデータを投入します。

```sh 
curl -XPUT 'http://localhost:9200/myvenues/venue/1' -d '{
    "name" : "Tokyo St",
    "tag" : ["station", "train"],
    "pin" : {
        "location" : {
            "lat" : 35.68,
            "lon" : 139.76
        }
    }
}'

curl -XPUT 'http://localhost:9200/myvenues/venue/2' -d '{
    "name" : "Nagoya St",
    "tag" : ["station", "train"],
    "pin" : {
        "location" : {
            "lat" : 35.17,
            "lon" : 136.88
        }
    }
}'
```

### 3. 位置情報で検索する

#### 位置＋距離

緯度/経度:35.6/139.8 から 20km 周囲にあるデータを検索します。

```sh 
curl -XPOST 'http://localhost:9200/myvenues/venue/_search' -d '{
    "query": {
        "filtered" : {
            "query" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_distance" : {
                    "distance" : "20km",
                    "venue.pin" : {
                        "lat" : 35.6,
                        "lon" : 139.8
                    }
                }
            }
        }
    }
}'
```

##### 結果

Tokyo St だけがヒットしました。

```javascript 
{"took":0,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":1,"max_score":1.0,"hits":[{"_index":"myvenues","_type":"venue","_id":"1","_score":1.0, "_source" : {
    "name" : "Tokyo St",
    "pin" : {
        "location" : {
            "lat" : 35.68,
            "lon" : 139.76
        }
    }
}}]}}
```

#### 範囲(矩形)

左上:35.2/136.8 〜 右下:35.1/136.9 にあるデータを検索します。緯度は上(北)の方が値が大きくなるので、上下関係に注意が必要です。

```sh 
curl -XPOST 'http://localhost:9200/myvenues/venue/_search' -d '{
    "query": {
        "filtered" : {
            "query" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_bounding_box" : {
	                "venue.pin" : {
	                    "top_left" : {
	                        "lat" : 35.2,
	                        "lon" : 136.8
	                    },
	                    "bottom_right" : {
	                        "lat" : 35.1,
	                        "lon" : 136.9
	                    }
	                }
	            }
            }
        }
    }
}'
```

##### 結果

Nagoya St だけがヒットしました。

```javascript 
{"took":0,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":1,"max_score":1.0,"hits":[{"_index":"myvenues","_type":"venue","_id":"2","_score":1.0, "_source" : {
    "name" : "Nagoya St",
    "pin" : {
        "location" : {
            "lat" : 35.17,
            "lon" : 136.88
        }
    }
}}]}}
```

#### 範囲(多角形)

任意の多角形領域にあるデータを検索します。

ここでは GeoJSON 互換の記述方式で書いてます。経度が先なので注意。
今までのような lat: lon: の配列でもかけますが、 GeoJSON 便利なので。

```sh 
curl -XPOST 'http://localhost:9200/myvenues/venue/_search' -d '{
    "query": {
        "filtered" : {
            "query" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_polygon" : {
	                "venue.pin" : {
	                	"points" : [
	                		[139.7, 35.7],  // 経度が先！
	                		[139.8, 35.7],
	                		[139.8, 35.6],
	                		[139.7, 35.6],
	                		[139.7, 35.7]
	                	]
	                }
	            }
            }
        }
    }
}'
```

##### 結果

Tokyo St だけがヒットしました。

```javascript 
{"took":0,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":1,"max_score":1.0,"hits":[{"_index":"myvenues","_type":"venue","_id":"1","_score":1.0, "_source" : {
    "name" : "Tokyo St",
    "pin" : {
        "location" : {
            "lat" : 35.68,
            "lon" : 139.76
        }
    }
}}]}}
```

(ポリゴンの座標群が、右回りじゃないとダメかな？と思って恐る恐る左回りにしてみたら、問題なく検索できました！)

## まとめ

最初 ``geo_point`` を明示的に指定しないといけないのに気づかなくてしばらくハマりましたが、それ意外はすんなりと動きました。

機能を試しただけでパフォーマンスなどは計測できていませんが、なんか使えそうな気はします。

位置情報関係の情報を探したい時は、公式サイトの GUIDE

* [Reference Guide | Elasticsearch](http://www.elasticsearch.org/guide/)

の検索バーで 「geo」で検索すると、有用な情報が得られます。

## 参考

* [Open Source Distributed Real Time Search & Analytics | Elasticsearch](http://www.elasticsearch.org/)
* [Geo Location And Search | Blog | Elasticsearch](http://www.elasticsearch.org/blog/geo-location-and-search/)
* [Spatial Search ElasticSearch tutorial - ElasticSearch Tutorial.com](http://www.elasticsearchtutorial.com/spatial-search-tutorial.html)
* [ElasticSearch geo distance filter with multiple locations in array - possible? - Stack Overflow](http://stackoverflow.com/questions/16113439/elasticsearch-geo-distance-filter-with-multiple-locations-in-array-possible)