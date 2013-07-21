---
layout: post
title: "Javascript 製のチャート描画ライブラリのメモ"
date: 2013-07-17 19:39
comments: true
categories: [javascript, HTML5]
---
Javascript 製のチャート描画ライブラリをいくつか調べたので備忘録として残しておく。
<!--more-->
やりたいのは、ストリーミングで次々やってくるデータをリアルタイムに表示する事。しかも順方向だけじゃなくて過去方向にも戻りたい。

## smoothiecharts

http://smoothiecharts.org/

シンプルで、使うのも簡単。が、逆再生ができるのかよく分からなかったので保留。

## Cubism.js

http://square.github.io/cubism/

言わずと知れたビジュアライゼーションライブラリ [D3](http://d3js.org/) のプラグイン。
なんか見た目がクール。
あまり突っ込んで調べてないので、要件を満たすかは不明。
ちなみにモバイル決済の Square によるオープンソースプロジェクト。
D3 自体でもいろいろなチャート描画ができるが、なんか勝手に Fat なイメージを持ってる。

## Flot

http://www.flotcharts.org/

こちらは jQuery のプラグイン。

使い方が簡単で、配列を描画させてるだけだったので、配列操作で逆再生にも対応できそう。

## Google Chart

https://developers.google.com/chart

大御所。
なんか "Connect to your data in realtime" って謡ってるので、できそうな感じもするが試してない。これも高機能であるが故にレスポンス大丈夫かなあと勝手に思っている。
 

## amCharts

https://amcharts.zendesk.com/entries/22592917-Creating-charts-with-real-time-data

amCharts というプロダクトで、リアルタイムなチャートが実現できる模様。参考程度に。

## その他

やりたいことは株価チャートに近いのでそっち方面で探すとたくさんありそうだけど、株価に特化しちゃってて機能過多＆使いづらい感。

## 参考

タイムリーにも同じようなまとめをしてくださってる方が居たのでメモ

* [グラフ描画に良さげなJavaScriptライブラリ - Qiita](http://qiita.com/hurutoriya/items/727296839a2ec638fdc4)