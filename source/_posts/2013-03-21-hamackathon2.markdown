---
layout: post
title: "ハマッカソン2 - #hamackathon に参加してきました"
date: 2010-12-6 7:02
comments: true
categories: [Android, 勉強会, GeoHex]
---
2010/12/4(土)、2回目の開催となる 浜松ハッカソン（ハマッカソン）、テーマがモバイルと聞いて Android  で参加してきました。
<!--more-->

<a href="http://www.flickr.com/photos/bigmac/5234115832/" title="Untitled by mackato, on Flickr"><img src="http://farm6.staticflickr.com/5283/5234115832_748685714b.jpg" width="500" height="332" alt="Untitled"></a>

* 浜松ハッカソン公開Wiki
* 第2回 浜松ハッカソン(ハマッカソン2) 開催レポート - Blog on AIRS
* ハマッカソン２に参加 - あきそふと開発日記

当日の Android チーム（というか自分）のレポです。

他の2チーム（HTML5チーム、 iPhoneチーム）は、チーム開発でしたが、Androidチームは一匹狼が多いのか銘々で好き勝手にアプリを作るという感じでした。

んで自分は、以前から「こんなのあったらイイな～」という俺得アプリを作りました。

###アプリ名： 
自宅から離れると勝手にマナーモードになるアプリ

###概要：
自宅でケータイをマナーモード状態で放置して重要なTELに気づかない。
または、会社で恥ずかしい着信音が鳴り響いてひんしゅくを買う。
ので、エリアに応じてマナーモードを ON/OFF する（+α）ようなアプリが欲しいなーって事で作りました。

 

既に同種の（というかまんまの）アプリがあるというのに！
作者様に敬意を表する意味で、ここで宣伝させていただきます。

* [AutoSetter (for 1.6) 場所によって端末の設定を自動で切替える 【300円】 | アンドロイダー](http://androider.jp/?p=21377)


まあなんとか妥協に妥協を重ねて時間内にデモれるくらいのアプリができ、プレゼンさせてもらった所、なかなか好評価を頂きまして、 Androidチームとして優勝できました！

写真だけじゃなんのこっちゃ分かりませんがこんなアプリです。

<a href="http://www.flickr.com/photos/bigmac/5234111922/" title="Untitled by mackato, on Flickr"><img src="http://farm6.staticflickr.com/5043/5234111922_286d7c8003.jpg" width="500" height="332" alt="Untitled"></a>

しかしまあ、この妥協具合は無いわ。最初「できる」と思ったのが恥ずかしいくらい。
どんな感じかというと。。。
 

### 当初予定
1. 地図アプリを起動したら地図を GeoHex を表示する。
2. マナーモードを ON or OFF するエリアを選択する。
3. 現在位置を取得する。GPS は電池消耗が激しいので WiFi か3G基地局から。
4. エリア内に現在位置が入ったら Intent を発行する。
5. 精度を含む現在位置が複数Hexにヒットした場合は、面積が一番大きいHexを採用する。
6. 受信側アプリで Intent を受け取ったらマナーモードを ON or OFF する。
7. 地図アプリは、タスクキラーに備えて AlarmManager で 5分置きとかに起動するようにする。
8. マナーモードON/OFFだけじゃなくて他にも何かできたらいいな。

### 実績

* 1. 2. は、事前に作って GitHub に登録してあった。つまりフライングギリギリ（汗
けど、[GeoHex](http://geogames.net/labs/geohex) は譲れなかったしね。 
* 3. は、GPS のみに。しかも通常 LocationManager を使うところを GoogleMap API の MyLocationOverlay で済ませた。
* 4. は Intent 発行でなく、マナーモード ON/OFF 処理を直書き（大汗
* 5. は 「GPS だから無視していっか」と妥協。
* 6. は Intent 使ってないので当然ボツｗ
* 7. は 「デモには関係ないから」 とスルー（滝汗
* 8. はデモ効果があるのでマジメに考えたｗ マナーモードON/OFF だけじゃなく、メディアボリュームをゼロにしたり、バイブレータを振動させたり、メーラーを起動したりした。

マトモに実装したのは、1. 2. 8 だけというｗｗｗ
いーんですよ、「機能を削ってでも納期を守る」 のがオトナですからｗ
 

Androidチームの成果物は、こうしてできた私のアプリと、 [@mackato](http://twitter.com/mackato) さんの 「ケモノカメラ」 、[@kiwofusi](http://twitter.com/kiwofusi) さんの 「ワンセグでTwitter（途中まで）」、 [@w1mvy](http://twitter.com/w1mvy) さんは加速度センサ、 [@natsumesou](http://twitter.com/natsumesou) さんは傾きセンサを使ったアプリでしたが惜しくも完成に至らず。。。
 

発表後の審査投票では、[@hide621](http://twitter.com/hide621) さん（なんと私と同じ校区に済んでいるのです）を始めとする iPhone チームの 「通信対戦型の時限爆弾アプリ」 と同数票となり、@mackato さんの 「ケモノカメラ」 を使った同点決勝で見事、私が勝利し、Androidチームの優勝となりました♪
 

優勝商品はメンバで山分けし、私は Google Developper Day 2010 の Tシャツ をもらいました。
さすが Google だセンスがイイぜ！
 

その後は懇親会。

<a href="http://www.flickr.com/photos/bigmac/5234117260/" title="Untitled by mackato, on Flickr"><img src="http://farm6.staticflickr.com/5122/5234117260_109e8293c6.jpg" width="500" height="332" alt="Untitled"></a>

懇親会では、10歳以上も歳の差のある[静岡大学情報学部 佐藤哲也研究室（通称 おめでてー研 というらしい）](http://tai.ia.inf.shizuoka.ac.jp/index.php?action=pages_view_main&page_id=43) の学生さん方々と相席になり、オジサン何話せばいいの？状態でしたが、

* iPhone アプリでガッポガッポな話
* Windows アプリでもガッポガッポな話
* フリーランスで悠々自適な話
* おめ研の就職先＆志望先にｳｯﾋｮｵｰな話
* 豊橋カレーうどん は浜松では誰も知らない話

などで盛り上がり、とても楽しく過ごせました。
 

次回は、

> 次回は3月前半の予定です。テーマとか要望があればTwitterで [@hamackathon](http://twitter.com/hamackathon) まで。
> via [blog.airs.co.jp](http://blog.airs.co.jp/2010/12/06/hamackathon-20101204.html)

とのことですので、また参加したいと思います。テーマは何がいいかな？
浜松近郊、また東海地方のエンジニアの皆様、ご参加されると楽しいと思います。

今回の私の成果物は、

[GitHub - amay077 - FollowerMap の hamackathon ブランチ](https://github.com/amay077/FollowerMap/tree/hamackathon) に置いてありますが、あまりにもやっつけ感ありありなので、近々マシにします。リポジトリ名が？なのはご愛嬌。
アプリとしてももっと育てていきたいので、継続的に開発していきたいと思います！

主催の [株式会社エアーズ @mackato](http://twitter.com/mackato) さん、参加された皆様、楽しい時間をありがとうございました。
 

####P.S.
[#GeoHex](http://geogames.net/labs/geohex) の布教活動しましたよ！ [@sa2da](http://twitter.com/#!/sa2da) さん！