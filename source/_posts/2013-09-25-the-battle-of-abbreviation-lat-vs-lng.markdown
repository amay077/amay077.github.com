---
layout: post
title: "Longitude の略し方。lng 派と lon 派の終わらない争い"
date: 2013-09-25 21:08
comments: true
categories: [Geo, GoogleMapsAPI]
---
地図上での位置は「緯度経度」で表します。英語だと「latitude, longitude」ですね。
これらの単語、コーディングする際は短縮したいわけです。latitude は ``lat`` で全会一致です。問題は longitude 。
<!--more-->
業界？の中では、longitude の略し方についての議論が度々沸き起こります。

例えば、、、

初めは、タイトルの議論で推移していましたが、次第に略し方の議論に。。。


* [latitudeとlongitude，どっちが緯度でどっちが経度？ - Togetter](http://togetter.com/li/85864)

``lon`` 派の方の意見。

<blockquote class="twitter-tweet"><p>そういえば緯度経度のうち経度（longitude）をlong, lon, lngなどと書く場合があるんだけど、longは長整数型のために予約語になってる場合があるのと、lngだと「イング」と読めてしまうからという理由でlonにしてる。</p>&mdash; Masaki Ohashi (@ohashimasaki) <a href="https://twitter.com/ohashimasaki/statuses/308895891308294144">March 5, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

``lng`` 支持？な方々

<blockquote class="twitter-tweet"><p>longitudeの省略形を何にするか迷ってる。lon or lng 今のところlngが優勢。</p>&mdash; 本城 博昭 (@honjo2) <a href="https://twitter.com/honjo2/statuses/28848028079">October 27, 2010</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p>最近Google Maps APIを触っていたので、CHINTAIの地図で経度(longitude)のパラメータがlngじゃなくてlonなのが新鮮に思えた</p>&mdash; Ryusuke SEKIYAMA (@rsky) <a href="https://twitter.com/rsky/statuses/25312333134">September 23, 2010</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

振り回されてる方々

<blockquote class="twitter-tweet"><p>lngとlonで５分ハマったー</p>&mdash; ばん↓どう↑さん↓ (@netartjp) <a href="https://twitter.com/netartjp/statuses/379923672632266752">September 17, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p>lonとlngのミスで1時間無駄にした</p>&mdash; カメキチ (@kamekiti) <a href="https://twitter.com/kamekiti/statuses/186149216492589056">March 31, 2012</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p>YahooのローカルサーチAPIのパラメーターの経度情報って、「lon」から始まる値なんですね。他の多くが「lng」な気がするので、始め何でエラーが出るのは分かりませんでした。ということで、ほぼ出来たピョン吉。</p>&mdash; 星野邦敏 (@khoshino) <a href="https://twitter.com/khoshino/statuses/149640555092131842">December 22, 2011</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

おいマジか Yahoo!（笑

<blockquote class="twitter-tweet"><p>「Y.LatLngオブジェクトのメソッドはlat、lngですが、ローカルサーチAPIのパラメータ名はlat、lonなので注意してください」YOLPで挑戦～「マクドナルドはどこだ」アプリをHTML5で作る！ <a href="http://t.co/QX8yiyvT">http://t.co/QX8yiyvT</a></p>&mdash; NI-Lab. (@nilab) <a href="https://twitter.com/nilab/statuses/189537270347472896">April 10, 2012</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


とこのように、一つの企業内でも「揺れ」が生じてしまう程度にはバラバラな感じです。

## 調べてみた

で、デファクトスタンダードはどれなのよ？をいろんな地図に関する Web API の仕様から一覧化してみました。

### Lng 派

Google は神！Google 先生について行きます！

* Google - https://developers.google.com/maps/documentation/javascript/reference?hl=ja#LatLng
* Mapion - http://labs.mapion.co.jp/api/asdoc/index.html
* ロケタッチ - http://tou.ch/developer/api_all?uri=spots%2Fsearch
* Leaflet - http://leafletjs.com/reference.html#latlng
* Yahoo(地図API) - http://developer.yahoo.co.jp/webapi/map/openlocalplatform/v1/js/#index6-2
* mapquest - http://www.mapquestapi.com/staticmap/#getmap

### Lon 派

頭から３文字取ったら普通これだろjk

* Yahoo(ローカルサーチ) - http://developer.yahoo.co.jp/webapi/map/openlocalplatform/v1/localsearch.html
* OpenStreetMap - http://wiki.openstreetmap.org/wiki/API_v0.6#Create_a_new_note:_Create:_POST_.2Fapi.2F0.6.2Fnotes
* OpenLayers - http://dev.openlayers.org/docs/files/OpenLayers/BaseTypes/LonLat-js.html ※LonLat！
* 電子国土Web - http://portal.cyberjapan.jp/site/mapuse4/#zoom=5&lat=35.99989&lon=138.75&layers=BTTT
* Elasticsearch - http://www.elasticsearch.org/guide/reference/mapping/geo-point-type/
* MongoDB - http://myadventuresincoding.wordpress.com/2011/10/02/mongodb-geospatial-queries/
* ゼンリンデータコム(いつもNavi) - http://support.e-map.ne.jp/manuals/V20/index.html?%A5%AF%A5%E9%A5%B9%A5%EA%A5%D5%A5%A1%A5%EC%A5%F3%A5%B9%2FZDC.LatLon
* 簡易逆ジオコーディングサービス / Finds.jp - http://www.finds.jp/wsdocs/rgeocode/index.html.ja#PARAMS
* ジオどす - http://geodosu.com/
* モバイラーズオアシスAPI - http://oasis.mogya.com/blog/API
* GeOAP(東京ガス) - http://dev.geoap.jp/GeOAP_Course/GeOAP_Trial.asmx?op=CourseLineOfLLToLL
* はてなココ - http://c.hatena.ne.jp/s/nearby?lat=36.2648177777778&lon=137.910003611111
* Nokia - http://developer.here.com/rest-apis/documentation/enterprise-map-image

### Long 派

「〜itude」は同じなんだから、それより前の部分を省略形にすべきだろ（でっち上げの根拠ですｗ

* LatLongLab(Yahoo) - http://latlonglab.yahoo.co.jp/
* はてなココ - http://developer.hatena.ne.jp/ja/documents/coco/apis/v1/spots#spots

### Longitude(略さない)派

こんなに迷うなら、いっそ省略形など要らぬ！

* Apple - https://developer.apple.com/library/ios/documentation/CoreLocation/Reference/CoreLocationDataTypesRef/Reference/reference.html
* Bing Maps - http://msdn.microsoft.com/en-us/library/gg427612.aspx
* Facebook -  https://developers.facebook.com/docs/reference/fql/location_post
* Amazon - https://developer.amazon.com/sdk/maps/api-reference.html
* Evernote - http://dev.evernote.com/doc/reference/Types.html#Struct_NoteAttributes
* Path - https://path.com/developers#tags

### 緯度経度ペアに名前付けちゃう派

だってペアじゃないと意味ないじゃん？

* Twitter(geocode) - https://dev.twitter.com/docs/api/1/get/search (緯度、経度)
* Foursquare(ll) - https://developer.foursquare.com/docs/venues/search (緯度、経度)
* カーリル図書館情報取得API(geocode) - http://calil.jp/doc/api_ref.html (経度、緯度)

### X/Y と同じじゃん派

所詮座標でしょ？(ﾎｼﾞﾎｼﾞ

* Solr(pt) - http://docs.lucidworks.com/display/solr/Spatial+Search 
* Oracle Spatial - http://otndnld.oracle.co.jp/document/products/oracle10g/102/doc_cd/appdev.102/B19243-02/sdo_cs_concepts.htm 
* PostGIS - http://postgis.net/docs/manual-2.1/using_postgis_dbmanagement.html#RefObject
* PlaceEngine - http://www.placeengine.com/doc/tut

## まとめ

数では ``lon`` 派が優勢。しかし開発者の目に触れる機会では Google が居る ``lng`` の方が多いのかも。
MS、Facebook など、割と巨大企業が 略さない を選択してるのも面白いですね。

ペアに名前を付けるやり方は、座標の順番が 緯度→経度 なのか、経度→緯度 なのか揺れるのであまり好きじゃないですね。

XY と同じ概念で扱うのは、そもそも地図以外での利用を想定していたり、地図でも投影されたXY座標を想定しているものが多いですが、XY と LatLon って、これまた混乱するんですよね。Lat って Y なの？順番逆なの？って。

個人的には 略さない派 だったのですが、そうは言っても短く書きたいので、最近は ``lon`` 派です。理由は、

**「オレオレ緯度経度クラスを使いたいが、Google に ``LatLng`` を取られちゃってるから」** 

クラス名が衝突すると、いろいろ面倒だし、クラス名に Prefix 付けるのもダサいし。。。

というわけで、終わらない争いと知りつつまとめてみました。
上記に載っていない情報があったら教えてもらえると嬉しいです。