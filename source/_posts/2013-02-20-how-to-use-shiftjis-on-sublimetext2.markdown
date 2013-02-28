---
layout: post
title: "SublimeText2 で Shift-jis を使う"
date: 2012-10-20 19:19
comments: true
categories: [SublimeText2]
---
SublimeText2 は大変便利なテキストエディタですが、shift-jis に対応していないのが玉にキズでした。

<!-- more -->

![mojibake](https://dl.dropbox.com/u/264530/qiita/shiftjis.png)

そんな時、@umibose さんのツイートが！

>Sublime Text 2でSJISもいける！https://t.co/XxCrfPd0
>
M.Hayakawaさん (@umibose) [10月 11, 2012](https://twitter.com/umibose/status/256247651710951424)

ふむ、ConvertToUTF8 というものできるのか。というわけで早速試してみました。

以下手順。

##1. Sublime Package Control の導入
[Installation – Sublime Package Control – a Sublime Text 2 Package Manager by wbond](http://wbond.net/sublime_packages/package_control/installation) の手順。

まず、SublimeText2 にパッケージコントロールシステムを導入します。(既に入っていればスキップ)

SublimeText2 を起動し、 ```shift + ctrl + @``` (ctrl+`) を押すと、画面下部にコンソールが表示されます。(Win版のショートカット知りません)

ここに、上記リンクにある import〜 の一文を貼り付けて Enter！

なんだかインストールが終わった感じなら Sublime を再起動します。

##2. ConvertToUTF8 の導入
[ConvertToUTF8 - Github](https://github.com/seanliang/ConvertToUTF8) の手順。

```shift + command + p```(Win版はry) を押すと、Command Palette が開くので、そこで ```Package Control: Install Package``` を選択。（ある程度入力するとサジェストされます）

次に出てくるポップアップ(画面名分からん)で ```ConvertToUTF8 ``` を選択。

インストールが終わったら Sublime を再起動すれば…

![choice](https://dl.dropbox.com/u/264530/qiita/choise_charcode.png)

お、なんか文字コード選択が出た（でない場合もある？）
そして…
![choice](https://dl.dropbox.com/u/264530/qiita/shiftjis_correct.png)

おっしゃー！
これであとは日本語入力時の挙動が直ってくれたら最強エディタですね。

[@umibose](https://twitter.com/umibose) さん、ありがとうございます！