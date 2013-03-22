---
layout: post
title: "第1回 名古屋Android勉強会に参加してきました"
date: 2011-01-17 8:38
comments: true
categories: [Android, 勉強会]
---
1/15（日） に行われた Android の初心者向け勉強会に参加してきました。
<!--more-->
[第1回 名古屋Android勉強会 : ATND](http://atnd.org/events/11259)

自分は昨年の9月頃から Android をぼちぼちと仕事と趣味でやってたので、今回参加してよいものが迷ったのですが、参加して良かったです。やはり我流は基本がいくつも抜け落ちている。やっぱり基礎は大事ですね。講師の有山さん( @keiji_ariyama ) の説明も分かりやすかったですし、実体験を交えて話される内容は説得力がありました。

勉強会の内容は、以下をご覧頂くとして－

* [nagoya-tsubu on USTREAM](http://www.ustream.tv/channel/nagoya-tsubu) ※2011/1/14~ のやつ
* [Togetter - 「#tsubu 第1回Android名古屋勉強会 2011/01/15」](http://togetter.com/li/89590)

個人的な発見だった点を備忘録してみます。

* Android を生業としてる人は AVD（エミュレータの事）を10種類以上用意している
	* いろいろな環境でテストされているんだなーと。 
* AVD を起動する時に 「Wipe User Data」 というオプションがあるのだけど、その意味が 「AVD をファクトリーリセット（作成時に戻す）」だと分かった。
	* こちらは twitter で [@androidzaurus](http://twitter.com/androidzaurus) さんに教えていただきました。ありがとうございます。 そして twitter すげェ！ 
* 多言語対応。文字列などのリソースは言語毎に res/values-xx に入れる。日本語なら res/values-ja 。 Android 端末側の言語設定に準じた res/values-xx が使われ、該当するものが用意されていないと res/values が使われる。 [@cyberspacefarm](http://twitter.com/cyberspacefarm) さんに補足いただきました。ありがとうございます。そして twitter すげェ！(2回目)
 
* 画面（Activity の増やし方）1. Activity を継承したクラスを作る 2.レイアウトファイル(xml)を作る 3. 1 の onCreate を実装する 4. AndroidManifest.xml に追記する。
	* 4. を知らなかったです。以前、画面遷移のサンプル作ろうとして挫折してた…
 
* 画面のレイアウトについて。 RelativeLayout 使おうぜ。
	* 難しそうなんだよなー。RelativeLayout …
 
* UnitTest について。Activity, Service, ContentsProvider に特化したテストクラスが用意されている。
	* 素直に知りませんでした。 Service のテストは便利そう。Activity のテストは、講師の有山さん曰く「面倒すぎてやってない」、うん納得。
 
* UnitTest の苦手なところ、「開発者の意図しない ”新しいバグ” を発見すること。」、その為に行うのが MonkeyTest 。ただし過信してはいけない。
	* 大いに納得。

こんな感じで勉強会終了。最後の方のテストの話は、 NUnit とかで単体テストを経験してたのでなんとかついていけましたが、会場の方々は結構しんどかったのかも。時間も 13:00～18:00 と長丁場だったので、さすがに疲れましたね。

さて、その後は、メイン(?)の懇親会へ。

懇親会では、 お隣りにさせて頂いた [@leibun](http://twitter.com/leibun) さん始め、[つ部](http://groups.google.co.jp/group/android-nagoya-tsubu) の一部の方々とお知り合いになることができましたし、 twitter で度々絡んでくださってた [@katono123](http://twitter.com/katono123) さんともリアルにお話できてとても楽しかったです。

是非とも次回の つ部の会合(?) に参加してみたいと思いましたし、アプリを作ってみたいと思いました。

最後に、講師の有山さん、主催・共催して頂いたつ部・日本Android東海支部のスタッフの皆さん、参加された皆さん、どうもありがとうございました。