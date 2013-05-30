---
layout: post
title: "Fused Location Provider を近くのショッピングセンターで評価してみた"
date: 2013-05-30 21:31
comments: true
categories: [Android, GoogleMapsAPI]
---
Android に新しく搭載された Fused Location Provider、これは GPS/WiFi/センサーを統合的に使ってその時ベストな位置を取得できるというもので、[Google I/O のデモ](http://www.youtube.com/watch?feature=player_detailpage&v=URcVZybzMUI#t=733s) が衝撃的だったので、自分でも試してみました。
<!--more-->
## 検証環境とか

* 場所 : イオン豊橋南店（ええ田舎ですが何か？
* 日時 : 2013/05/30 17:30頃
* 天候 : 小雨（曇天だったので GPS は捕まえにくかったかも）
* 端末 : au HTC J(!蝶) OS は 4.0.4
* GPS も WiFi も ON
* Android SDK Tools : rev.21.1
* Google Play service : rev.7
* FusedProvider の Priority : PRIORITY_HIGH_ACCURACY
* 位置取得間隔 : 5秒
* 姿勢 : スマホを常に左手で掲げて、見ながら歩きました。

## その1:駐車場〜1F〜駐車場

駐車場に車を停めて、歩いて店内に入場、店内をなるべくゆっくり直線的にぐるっと徘徊して、退店、駐車場を歩いて戻って来ました。

緑が実際の経路、青が Fused Provider の経路です。

<iframe width="425" height="350" frameborder="0" scrolling="no" marginheight="0" marginwidth="0" src="https://maps.google.co.jp/maps/ms?msa=0&amp;msid=206106708723125678709.0004ddec1d5240e6ba1d3&amp;brcurrent=3,0x6004d3ebe16cdde1:0xfb92e7477942b89b,0&amp;ie=UTF8&amp;t=h&amp;ll=34.709597,137.387786&amp;spn=0.001372,0.001735&amp;output=embed"></iframe><br /><small><a href="https://maps.google.co.jp/maps/ms?msa=0&amp;msid=206106708723125678709.0004ddec1d5240e6ba1d3&amp;brcurrent=3,0x6004d3ebe16cdde1:0xfb92e7477942b89b,0&amp;ie=UTF8&amp;t=h&amp;ll=34.709597,137.387786&amp;spn=0.001372,0.001735&amp;source=embed" style="color:#0000FF;text-align:left">FusedProviderの評価(イオン豊橋南店1F)</a> を表示</small>

## その2:駐車場〜1F〜2F〜ぐるっと〜1F〜駐車場

もう一度。
今度は入店してすぐエスカレータで2Fへ。2Fフロアをぐるっと回ってからエスカレータで1Fへ降りて駐車場へ戻って来ました。

<iframe width="425" height="350" frameborder="0" scrolling="no" marginheight="0" marginwidth="0" src="https://maps.google.co.jp/maps/ms?t=h&amp;brcurrent=3,0x6004d3ec8f3f5bf5:0x7b6f4e2f69453e37,1&amp;msa=0&amp;msid=206106708723125678709.0004ddec316197e38b987&amp;source=embed&amp;ie=UTF8&amp;ll=34.709546,137.388057&amp;spn=0.003479,0.004517&amp;output=embed"></iframe><br /><small>大きな地図で <a href="https://maps.google.co.jp/maps/ms?t=h&amp;brcurrent=3,0x6004d3ec8f3f5bf5:0x7b6f4e2f69453e37,1&amp;msa=0&amp;msid=206106708723125678709.0004ddec316197e38b987&amp;source=embed&amp;ie=UTF8&amp;ll=34.709546,137.388057&amp;spn=0.003479,0.004517" style="color:#0000FF;text-align:left">FusedProviderの評価(イオン豊橋南店2F)</a> を表示</small>

## 結果をみて

うーん、かいかぶり過ぎたか Fused Provider。

確かに GPS と WiFi をシームレスに扱ってくれているようですが、期待していた屋内での測位結果はちょっと残念でした。

I/O のセッションの中では、WiFi+Sensor を使ってると言っていたので、WiFi-AP の電波強度と加速度センサーの振れ具合で自律測位してくれるのかなあと思ったのですが、あまり自律測位が機能してないように見えます。

###  Accuracy は？

屋外だとだいたい 10m前後、屋内でも 20〜40m くらいの精度でした。
上図の幅が約200mですが、実際の位置よりもっと離れている感じがします。

Fused Provider を使うと、妙に Accuracy が小さい(精度の良い)値が得られるのですが、実際の位置がその精度が示す円の中にも入らないこともあり、それなら高い確率で実際の位置を包括する（精度の悪い）WiFi or 基地局測位 の方が有用では？とも思えました。

また、施設内に WiFi-AP が何個あったとかの細かい調査はしていません（たぶん５〜６個）が、都会の繁華街とか地下街の方が圧倒的に多いと思うので、また違った結果が出るものと思います。

### GPS の動きは？

今回は ``PRIORITY_HIGH_ACCURACY`` を使いました。これは GPS を使います。(``PRIORITY_BALANCED_POWER_ACCURACY`` は GPS を使いません［が、 ``ACCESS_FINE_LOCATION`` を付けないと精度が数kmレベルになります。］)

記録中の GPS アイコンの動きは、点滅 → 数秒後消える → また点滅 → GPSを捕捉したらつきっぱなし という感じで、屋外に移動すると GPS を捕まえる、屋内では度々トライするが捕まえられないのですぐ消える。という動きをしてました。

### 状態認識は貢献してるのか？

自律測位に状態認識(Activity Recognition)がもし使われているなら、持ちながらよりも、ポケットに入れっぱで歩いた方に最適化されてたかも知れないなあと思ったり。

[ActivityRecognitionClient を試した](http://amay077.github.io/blog/2013/05/18/getting-started-activity-recognition/) 時も、持ちながらよりも尻ポケに入れたまま歩いた方が認識されやすい気がしました。気だけですが。

## まとめ

* Fused Provider に期待しすぎないでください
* 実際の位置から（精度値を超えて）大きく外れることもあるのが困る
* パワーマネジメントはなんかやってるぽい（計測してないけど）