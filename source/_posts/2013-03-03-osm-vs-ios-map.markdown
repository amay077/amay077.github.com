---
layout: post
title: "OpenStreetMap と iOS6 の地図を比べてみました"
date: 2012-10-03 21:08
comments: true
categories: [OpenStreetMap, iOS]
---
####9.20追記:OpenStreetMap≒MapQuest な事など加筆修正しました

完全に乗り遅れた感がありますが、OpenStreetMap とiOS6の地図を比べてみました。

<!-- more -->

[OpenStreetMap](http://www.openstreetmap.org/) は誰でも編集できて自由に使える地図のWikipedia です。
(iOS6地図ガッカリ事件に際して、OpenStreetMap のデータに原因があるかのような報道がありましたが、[きっぱりと否定](http://www.osmf.jp/news/protestagainstnyt)しています。)

OpenStreetMap を使ったアプリを開発する方法はさまざまですが、ここでは、MapQuest社の開発キットを紹介します。

* [Apple iOS Maps API - MapQuest Developer Network](http://developer.mapquest.com/web/products/featured/apple-ios-maps-api)

またお約束ではございますが、この記事は、Yahoo さん、Mapion さんの完全なパクリでございます、ご了承ください！

* ヤフーさんのYOLPの記事は[こちら](http://blog.olp.yahoo.co.jp/archives/20120920_yolpios6.html)
* マピオンさんのネタ記事は[こちら](http://labs.mapion.co.jp/blog/labs/ios6.php) ☆(ゝω・)v

###OpenStreetMap と MapQuest社 について
OpenStreetMap は「地図データ」、MapQuest社はそのデータを利用してアプリやサービスを提供する企業です。OpenStreetMap の「◯年◯月◯日時点」のスナップショットを自社で運用して、デザインを調整し、Webサービスやアプリ、SDK として提供しています。

ここで紹介している地図画像は OpenStreetMap そのものではなく、「MapQuest の〜」です。起源は同じ、見た目も似ていますが、正確には異なることをご了承ください。OpenStreetMap 本家の地図はリンクから確認できるようにしてあります。

では、さっそく。

##東京スカイツリー
###iOS6:
![sky](https://dl.dropbox.com/u/264530/qiita/sky-ios.png)

###OpenStreetMap(MapQuest):[→本家](http://www.openstreetmap.org/?lat=35.71003&lon=139.81066&zoom=17&layers=M)
![sky](https://dl.dropbox.com/u/264530/qiita/sky-osm.png)

ちゃんとあるよスカイツリー。

##渋谷ヒカリエ
###iOS6:
![hikarie](https://dl.dropbox.com/u/264530/qiita/hirarie-ios.png)

###OpenStreetMap(MapQuest):[→本家](http://www.openstreetmap.org/?lat=35.659051&lon=139.70417&zoom=18&layers=M)
![hikarie](https://dl.dropbox.com/u/264530/qiita/hirarie-osm.png)

ちゃんとあるよヒカ…な、ない！
※本家にはちゃんとヒカリエありました！[hal_sk](http://qiita.com/items/03cf7e04cac886b84136#comment-055c7f54ebf92d3bd05b) さん、ありがとうございます。

##新宿駅周辺
###iOS6:
![shinjuku](https://dl.dropbox.com/u/264530/qiita/shinjuku-ios.png)

###OpenStreetMap(MapQuest):[→本家](http://www.openstreetmap.org/?lat=35.68926&lon=139.70072&zoom=17&layers=M)
![sinjuku](https://dl.dropbox.com/u/264530/qiita/shinjuku-osm.png)

Apple よ、これが線路だ！

##新東名高速道
###iOS6:
![tomei](https://dl.dropbox.com/u/264530/qiita/tomei-ios.png)

###OpenStreetMap(MapQuest):[→本家](http://www.openstreetmap.org/?lat=34.9941&lon=138.4068&zoom=12&layers=M)
![tomei](https://dl.dropbox.com/u/264530/qiita/tomei-osm.png)

最初に新東名を描いたのは誰でしょう？

##まとめ

いかがでしたか？

OpenStreetMap は、誰もが編集できる自由な地図です。日夜マッパーさん達が新しい道路や建物を描いてくださっています。そのため、情報量では、少なくともiOS6地図よりは優っています。(場所によっては Googleマップより詳細な地域も ex:[北朝鮮までカバー！地図のWikipedia「OpenStreetMap（OSM）」がすごい件 | ihayato.news](http://www.ikedahayato.com/index.php/archives/15493))

しかし地図デザインはというと、洗練されてはいません。
OpenStreetMap はデータであり、デザインは範疇ではない、と言えます(補足２へ)。

地図のデザインとは、色合い、線の種類や太さ、注記やアイコンの大きさ・配置、どんな情報を表示する／しないかの取捨選択、時には人が見やすいようなディフォルメを加えたり、と職人さんの苦労の賜物です。

* [これがニッポンの地図づくり！地図職人のこだわりお見せします - YOLP](http://blog.olp.yahoo.co.jp/archives/20120928_chukihaichi.html)
* [YOLP地図デザインスタッフが語るスマートフォン専用スタイル地図の5つの魅力 - YOLP](http://blog.olp.yahoo.co.jp/archives/20120720-mapdesign.html)
* [グッドデザイン賞をマピオンの地図が受賞：マピオン](http://www.mapion.co.jp/topics/gooddesign/)

iOS6マップは、「スカスカ」「注記がおかしい」「位置がズレてる」などが指摘されていますが、それが解決されたとしても、「地図デザイン力」で、Yahoo、Mapion、Google などの他社に追いつけるでしょうか？

テクノロジーだけでも、コンテンツだけでも、両方揃っていてもダメ。
お金では買えない価値がある。
買えるものは・・・**企業じゃね？**

というわけで１年後、iOS6マップと地図業界がどうなっているかが楽しみです。

####補足
* 本記事内で紹介したiOS6地図画像は、マピオンさんの記事から拝借しました。問題があったら言ってください。
* 同OpenStreetMap地図画像は、[MapQuest社のAndroidアプリ](https://play.google.com/store/apps/details?id=com.mapquest.android.ace&hl=ja)からキャプチャしたものです。[iOS版](http://itunes.apple.com/us/app/mapquest/id316126557?mt=8)は日本では使えませんでした、なんだよぅ。
* Qiita でこんな事書いていいのかしら？SDKに触れてるから、いいよね？問題があったら言ってください(またか)

####補足２
OpenStreetMap を使った地図サイトの「見た目」は、地図データを描画する "マップレンダラー" に何を採用するかで大きく異なります。openstreetmap.org のサイトでは [Mapnik](http://wiki.openstreetmap.org/wiki/JA:Mapnik) というライブラリが採用されていますが、他にも[さまざまなもの](http://wiki.openstreetmap.org/wiki/Rendering)があるようです。
しかし "マップレンダラー" は地図に適用するスキンのようなもので、よく目的にあった地図を作るには地図データそのものを「加工」する必要があるのではないかと思っています。(この辺、ちょっと勉強不足)

OpenStreetMap については、 [dkastl@github](http://qiita.com/items/03cf7e04cac886b84136#comment-49a56e8d30273ad833e4) さんから良いコメントを頂いたので、全力で和訳してみました。

>OpenStreetMap の特長は、問題や間違っている情報を見つけた時に、すぐにそれを修正したり新しいオブジェクトを追加できることです。
>そして、あなたが次に訪れた場所は、地図に載るでしょう。(^^)
>
>そして、高速道路や新しい建物のような人気のスポットに間違いを見つけたらあなたはラッキー、それを(修正して)貢献し、賞賛を受けられます。

OpenStreetMapの創始者であるスティーブ・コースト氏は、先に日本で行われたOpenStreetMapの国際会議(SotM)でこう語っています。

> “LinuxがプロプライエタリなUNIXの品質を凌駕したように、Wikipediaがプロプライエタリな辞書を超えたように、OpenStreetMapの地図の情報は、将来、プロプライエタリな地図を超えるだろう

via [SotM 2012 Tokyo レポート - “地図のWikipedia”OpenStreetMapの国際会議が日本で初開催：ITpro](http://itpro.nikkeibp.co.jp/article/COLUMN/20120910/421702/?ST=cloud&P=3)

そんな OpenStreetMap の未来に期待しつつ、今日も私は[のんほいパーク](http://www.openstreetmap.org/?lat=34.72014&lon=137.4328&zoom=17&layers=M)を描き続けるのです。