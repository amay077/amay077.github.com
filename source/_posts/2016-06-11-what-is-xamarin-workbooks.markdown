---
layout: post
title: "Xamarin Workbooks とかいうやつ"
date: 2016-04-29 00:00:00 +0900
comments: true
categories: [iOS, C#, Xamarin, Android, Markdown] 
---

[Xamarin Evolve 2016](https://evolve.xamarin.com/) が開催中されました。
<!--more-->
build での予告どおり、Xamarin.Android/iOS などがついにオープンソースになった、などのエキサイティングな発表のまとめは

* [【速報】Evolve 2016 で発表されたエキサイティング情報まとめ - Xamarin 日本語情報](http://ytabuchi.hatenablog.com/entry/evolve2016)

その Keynote でデモされていた Xamarin Workbooks というツールがなかなかすごいので紹介。

## なにこれ？

ひとことでいうと、

**Xcode の Playground みたいなやつ + Markdown**

まだ意味わかんないですね？

こういうことです。

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_workbooks_01.png)
![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_workbooks_02.gif)

* Markdown でドキュメントが書ける(このツール自体はリッチなエディタである)
* \`\`\`csharp〜\`\`\` で囲まれたコードブロックは、そのまま iOSシミュレータなどでインタラクティブに、Instant に実行できる。

上記の Workbook の実ファイルは、これ↓です。

* [Xamarin Workbooks を使ってみるテスト。 HowToUseMapKit.workbook で保存して Xamarin Inspector で File->Open してね。](https://gist.github.com/amay077/793b5df4aad0098ffe6d9c12a491ee9a)

みてわかる通りまんま Markdown ですね。

## Let's try!

しかも iOS だけじゃなく、Android, Mac, Windows(WPF) に全対応！
Mac でも Windows でも試せるみたいです。

* [Xamarin Workbooks - Xamarin](https://developer.xamarin.com/guides/cross-platform/workbooks/)

スタンドアロンなアプリとして実行可能なようなので、サクッと使ってみよう！
