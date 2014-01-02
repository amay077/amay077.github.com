---
layout: post
title: "NHK紅白の Android/iPhone アプリが .NET/Xamarin 製だったということ"
date: 2014-01-02 11:27
comments: true
categories: [Xamarin, Android, iOS, C#]
---
新年あけましておめでとうございます。
今年も Xamarin 推しで参ります、よろしくお願いします。
<!--more-->
2013年大晦日の紅白歌合戦、NHK が iPhone/Android 用のアプリを配信していました。

* [紅白アプリ｜第64回NHK紅白歌合戦](http://www1.nhk.or.jp/kouhaku/app/)
* [紅白で「イェーガー！」と叫ぶために曲を見逃さないiPhoneアプリ、NHK紅白](http://weekly.ascii.jp/elem/000/000/192/192769/)

なんとこのアプリ、Xamarin 製だったとのこと。
紅白あんまり興味なかったのでノーチェックでしたわー。

<blockquote class="twitter-tweet" lang="ja"><p>紅白アプリXamarinなんか</p>&mdash; ゆたか (@tmyt) <a href="https://twitter.com/tmyt/statuses/413092620567470080">2013, 12月 17</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" lang="ja"><p>iPhoneの紅白アプリ、MvvmCross使ってるってことは、Xamarinで作ってるってこと？！ <a href="http://t.co/cTWPz2cp9E">pic.twitter.com/cTWPz2cp9E</a></p>&mdash; 菊池紘 (@kikuchy) <a href="https://twitter.com/kikuchy/statuses/417977438597959680">2013, 12月 31</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

このツイートを RT した後、ソッコーで Android 版を入れてみましたら、確かにクレジットに MvvmCross やら ActionBarSherlock for Xamarin やらならんでいました。

## 開発してるのは…

開発は、[スレイプニル](http://www.fenrir-inc.com/) で有名なフェンリルさんのようですね。

* [フェンリル株式会社 | スマートフォンアプリ開発 実績 NHK 紅白](http://biz.fenrir-inc.com/application_development/casestudy_app/nhk_kouhaku.html)

2年連続で作っておられるようですが、おととしから Mono(MonoTouch/Mono for Android) 製だったのかな？いやスゴいです！

## 使われているライブラリ

せっかくなので、Android/iOSアプリ両方の著作権表示から、使われている OSS ライブラリを列挙してみます。([こちらのエントリ](http://nkzn.hatenablog.jp/entry/2013/12/30/010956)にインスパイアされました)
当然ですが、すべて .NET/Mono で動作するライブラリばかりです。(ActionBar と Nimbus を除く)

### MvvmCross

* https://github.com/MvvmCross/MvvmCross
* クロスプラットフォームMVVMフレームワーク。[こちらでも](http://qiita.com/amay077/items/c4227663b5a5e540dc13) 紹介しました

### Json.NET

* http://james.newtonking.com/json
* .NET/Mono で JSON を扱うための事実上標準ライブラリ

### SocketIO4Net.Client

* http://socketio4net.codeplex.com/
* WebSocket4Net と組み合わせて使うっぽい？ライブラリ

### WebSocket4Net

* http://websocket4net.codeplex.com/
* .NET で WebSocket 使うためのライブラリ。

### SuperSocket.ClientEngine

* http://clientengine.codeplex.com/
* ソケット通信用ライブラリっぽい。

### MvxSettings

* https://github.com/jamesmontemagno/Mvx.Plugins.Settings かな？
* 設定情報をストアするための、MvvmCross のプラグイン

### ActionBarSherlock for Xamarin

* http://components.xamarin.com/view/XamarinActionBarSherlock
* スライドメニュー(NavigationDrawer) を実現するライブラリ。そういえばちょっと変わったスライドメニューでしたね。

### AsyncOAuth

* http://neue.cc/2013/02/27_398.html
* C#/LINQ の神であらせられる [@neuecc](http://neue.cc/2013/02/27_398.html) さん作の 非同期OAuthライブラリ

### Nimbus

* http://nimbuskit.info/
* iOS の UIパーツがいろいろ拡張されてる的なライブラリ？Xamarin.iOS で Binding して使ってるのかなあ？

### Html Agility Pack

* http://htmlagilitypack.codeplex.com/
* HTMLパーサライブラリ

生放送のテレビ番組向けアプリということで、リアルタイム通信に注力された様子が、使用されたと思われるライブラリからも伺えます。

## これは強力すぎる Xamarin 導入事例ですね

日本の最も有名なテレビ番組のスマホアプリに Xamarin が使われていたというのは大きな導入事例になること必至です。
アプリの性質上、期間限定となる可能性もあります。できればこのまま公開しつづけて欲しいですが、Xamarin を上司や提案先に紹介されたい場合は、お早めに、またキャプチャを多く撮っておかれる事をおすすめします。

最後に、今年が Xamarin 普及元年とならん事を近くの神社にお祈りして、新年最初のエントリの締めとします。