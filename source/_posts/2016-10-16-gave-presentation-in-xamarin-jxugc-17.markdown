---
layout: post
title: "JXUGC #17 お前の Xamarin アプリを見せてみろ！ で発表してきました"
date: 2016-10-16 19:29:05 +0900
comments: true
categories: [Xamarin, 勉強会]
---

[JXUGC #17 お前の Xamarin アプリを見せてみろ！ - connpass](http://jxug.connpass.com/event/39470/) に参加、登壇してきました。

<!--more-->

[Xamarin DataPages](https://developer.xamarin.com/guides/xamarin-forms/datapages/) をネタにしましたが、今回は本当に準備が不足してすいませんでした。何かをでっちあげることしかできませんでした。

他のみなさんのアプリや発表、完成度が高くて脱帽です。

一応、資料はこちら。

<script async class="speakerdeck-embed" data-id="f7a39d6e6d5742a185231e1e4e1d8953" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

ソースもあります。(Xamarinのリポジトリから切り取ってきた DataPages 関連のプロジェクト付き)

* [amay077/MikawaMorningApp](https://github.com/amay077/MikawaMorningApp)

今回の参加は、たまには東京の人たちにも会いたいなーという動機、それから Xamarin.Forms.GoogleMaps の PR が目的でして。。。

DataPages にそこまで期待しているわけじゃないけど、標準機能でできることが少ないなら Xamarin.Forms.GoogleMaps みたく、Fork して機能拡張しちゃえばいいやと軽い気持ちでソースを除いてみたのですが、うーむ分からん。

``Xamarin.Forms.Pages.DataPage`` を継承したクラスに ``ListDataPage`` というものがあって、これは与えられたデータソースを元に一覧画面を生成してくれます。更に、行を選択すると詳細画面も表示してくれます。

ならば同じく ``DataPage`` を継承して ``MapDataPage`` を作ったら、一覧の代わりに地図にピン群が立つ画面を用意できるのでは？そんでピンをタップしたら詳細画面を表示できるのでは？と思ったのですが・・・。

``MapDataPage`` を用意することはできました。データソースを元に地図にピン群を立たせることはできました。

しかし、詳細ページを表示する方法が分かりませんでした。

もうちょっと詳しく言うと、``ListDataPage`` で行を選択したときに [``DataTemplate.CreateContent()``](https://github.com/amay077/MikawaMorningApp/blob/master/Xamarin.Forms.Pages/ListDataPage.cs#L46) を呼び出していて、その返値が詳細画面となる ``Page`` なのですが、 この ``DataTemplate`` がよく分からない。

``ListDataPage.DetailTemplateProperty`` への設定はいつだれが行っている？ DataTemplate は Xamarin.Forms.Core のソースなので、ちゃんとデバッグ環境作ればトレースできたかもしれません（だから Xamarin.Forms.Themes は関係ないかも知れない）が、ちょっと時間なく。。。

この辺りの仕組みが分かれば、詳細画面で、

* http: で始まる文字列には、ハイパーリンクを設定する
* 緯度経度だったら「地図」ボタンを表示する

などのカスタマイズもできると思うので、引き続きソースおっかけてみたいと思います。

JXUG イベント多すぎだろ！と思いつつ、数回に１回は行きたいなーと思っているので、またよろしくおねがいします。

最後に、「喫茶店で朝￥500円以上払ったらそれはモーニングサービスとは言わない」これはガチ。
