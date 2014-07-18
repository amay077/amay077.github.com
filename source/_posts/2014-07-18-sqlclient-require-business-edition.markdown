---
layout: post
title: "Xamarin で System.Data.SqlClient を使うには BUSINESS 版以上が必要です"
date: 2014-07-01 01:00
comments: true
categories: [Xamarin]
---
当方 Xamarin INDIE 版しか買えないしがない個人開発者です。
<!--more-->

Xamarin で ``System.Data.SqlClient.SqlCommand`` などを使ったプロジェクトをビルドしたら、ビルド時にこんなダイアログボックスが。

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_requires_business_edition_when_using_sqlclient_01.png)

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_requires_business_edition_when_using_sqlclient_02.png)

どうやら、特定のクラスを使用するには BUSINESS 版以上が必要なようです。

* [Xamarin でビルドを自動化するには Business 版以上が必要です](http://qiita.com/amay077/items/ab90c74e78dd87ba31fb)

といい、INDIE 版の制限事項が後出しで判明するのなんとかならないですかね。

今回は .NET のプロジェクトを Xamarin に移植する際に発覚したもので、特に ``SqlClient`` は使ってなかったので削除して解決しました。

Xamarin で ``SqlClient`` って何に使うんだろ？イントラ？あるいは SQL Server Compact が使える？まさかね。
