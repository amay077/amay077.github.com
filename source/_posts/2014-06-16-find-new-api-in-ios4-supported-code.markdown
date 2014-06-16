---
layout: post
title: "iOS4.x 対応のソースコードから「新しいAPI」を使っている箇所を探す"
date: 2014-05-14 15:26
comments: true
categories: [Objective-C, iOS]
---

　アプリを iOS4.x でも動作させたい場合、Xcode のプロジェクト設定で Deployment Target を "4.x" (4.3とか) に設定します。
一方で Base SDK は最新のものしか選べません、今だと "7.1"。
<!--more-->
　この状態だと、コード中で iOS5以降に追加された API を使っていると、iOS4.x端末では当然クラッシュします。

　Target を 4.x にしてるんだから、クラッシュしそうなコードがあったら Xcode が検出して警告して欲しいんですが、そういう機能はないみたいです。(実はあるのでしょうか？ Android だと警告どころかビルドエラーになるので、iOSアプリ開発は大変不便だな、と思ってしまいます。Obj-C は JavaScript みたいなもんだから仕方がない、のは分かりますが)

　 [OCLint](http://oclint.org/) という静的コード解析ツールを見つけましたが、機能をざっと見ても、APIバージョンをチェックするものはなさそうです。（試したことはありません）

　しかし「動かしてみないと分からない」のは不安すぎるので、なんとかして「新しいAPIを使っていないか？」をチェックする方法を考えて、行ってみました。

## 新しいAPIを使っている箇所を見つける方法

### A. iOS Developer Center に API の更新内容がまとめられたページがあります。

* [iOS 4.3 to iOS 5.0 API Differences](https://developer.apple.com/library/ios/releasenotes/General/iOS50APIDiff/index.html#//apple_ref/doc/uid/TP40011042)
* [iOS 5.1 to iOS 6.0 API Differences](https://developer.apple.com/LIBRARY/ios/releasenotes/General/iOS60APIDiffs/index.html)
* [iOS 6.1 to iOS 7.0 API Differences](https://developer.apple.com/LIBRARY/IOS/releasenotes/General/iOS70APIDiffs/index.html)


### B. これらのページから「変更のあった API のリスト」を抽出します。

具体的には以下のようなリストを作ります。
>Added vImageAlphaBlend_ARGB8888
>Added vImageAlphaBlend_ARGBFFFF
>…

この作業は自動化したいのですが、お試しなので手動でテキストエディタと **EXCEL** を駆使して作成しました。

### C. あとは、自分のソースコードに対して、順次 grep をかけるスクリプトを作ります。

修正リストの中には、 ``-[ALAsset setImageData:metadata:completionBlock:]`` のように名前付き引数だったりする API もあるので、正規表現でなるべくヒットするように置換します。

```sh find_new_api.sh
grep -nr "ALAsset.*setImageData.*metadata.*completionBlock:.*;" ./src/*
grep -nr "vImageAlphaBlend_ARGB8888.*;" ./src/*
grep -nr "vImageAlphaBlend_ARGBFFFF.*;" ./src/*
…
```

### D. このスクリプトを実行して何か出たら、そこが「新しいAPIを使ってる箇所」です。

## まとめ

### 注意点

* クラス自体が追加されているものも、メソッドの追加として探しているので、他のクラスと誤認することがあります。
* 正規表現が完全に正しいかよくわかりません（ソースコードに改行含む場合とか）


簡単で、不格好な方法ですが、十分に機能してくれますし、開発者の知識やテストだけに頼るのに比べれば随分と安心できます。

APIの更新情報を JSON か何かで提供してくれたら、もうちょっと楽なんですけども。。。

iOSアプリの開発者のみなさんは、どうやって iOS の下位バージョン互換性を担保しているのでしょうか？
