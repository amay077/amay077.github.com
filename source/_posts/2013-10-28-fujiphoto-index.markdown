---
layout: post
title: "富士フォトというアプリをリリースしました"
date: 2013-10-28 18:22
comments: true
categories: [Android, MA9]
---
富士山の撮影スポットを探して、そこへ案内するアプリです。
<!--more-->
[![img1](https://dl.dropboxusercontent.com/u/264530/qiita/fujiphoto_00.png)](https://play.google.com/store/apps/details?id=com.amay077.android.fujiphoto)

撮影スポットは、静岡県が提供している「ふじのくにオープンデータ」の「[富士山ビューポイント](http://open-data.pref.shizuoka.jp/htdocs/index.php?action=pages_view_main&active_action=multidatabase_view_main_detail&content_id=33&multidatabase_id=2&block_id=15#_15)」を使用しています。

以下のことができます。

* GPS を使って、現在地付近にある撮影スポットを検索
* スポットで撮影された富士山の写真を表示
* 現在地と撮影スポット、富士山の位置関係を地図で確認
* 撮影スポットと富士山付近の(雨)雲の状況を地図に表示
* 現在地から撮影スポットへ行くまでを AR で案内(Googleマップナビも使用できます)
	* AR は単純に富士山の方向を確認するのにも使えます。

## 使い方

### 近くの撮影スポット

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/fujiphoto_01.png)

### 撮影スポットの詳細

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/fujiphoto_02.png)

### ARナビ

田舎すぎるだろ…

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/fujiphoto_03.png)

## 技術的なアレ

このアプリで [MashUpAward9](http://ma9.mashupaward.jp/works/348) に応募してます。
使用した API は、

* [Yahoo! Android マップ SDK](http://ma9.mashupaward.jp/apis/216) の MapView と雨雲タイルと ARナビ
* [Yahoo! Open Local Platform](http://ma9.mashupaward.jp/apis/218) のリバースジオコーダ
* [datastore API](http://ma9.mashupaward.jp/apis/145)

です。本当は Googleマップの代わりに [Mapion](http://ma9.mashupaward.jp/apis/36) の「3D風地図」を使いたかったのですが、時間がなくてあきらめました。(ピンチズームとかを自前実装しなければならなさそうだったので)

「datastore API」 は [appiaries さんの BaaS](http://www.appiaries.com/jp/) のひとつの機能です。この BaaS は位置情報にも対応していて、範囲検索なども行ってくれるのですが、こちらも今回時間がなくて、他のコードを使いまわして自力実装しました。

データは、冒頭で紹介した、

* [富士山ビューポイント - ふじのくにオープンデータ](http://open-data.pref.shizuoka.jp/htdocs/index.php?action=pages_view_main&active_action=multidatabase_view_main_detail&content_id=33&multidatabase_id=2&block_id=15#_15)

を使っています。
日頃アプリを開発したり、アイデアを練ったりしていて、どうしても行き詰まるのがコンテンツです。隣県でこのようなコンテンツを公開してくれる事はとてもありがたく、ぜひ活用してみたいと思って作りました。

まだまだ不安定な感じ（メモリ不足でたびたび落ちる(汗)）ですが、ちまちまブラッシュアップしていきたいと思います。