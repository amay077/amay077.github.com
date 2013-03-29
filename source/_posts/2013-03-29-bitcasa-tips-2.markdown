---
layout: post
title: "Bitcasa は Mac の Terminal で操作できたよ"
date: 2013-03-29 21:05
comments: true
categories: [Backup, Bitcasa]
---
[Bitcasa を使っていて「おや？」と思ったこと](http://amay077.github.com/blog/2013/03/25/bitcasa-tips-1/) の続きというか対策というか。
<!--more-->
>あれ？コピーしたと思ったんだけどなあ
Finder でコピーして <中略> なんかファイル数が少ない事があったようななかったような。。。

という事があって、アップし損ねたファイルをアップロードしようとした時の話。
ファイルを一つずつ目視確認してくのツラいので、Finder で全ファイル選択して Bitcasa 側へコピー。その後、既に存在してるファイルはスキップするか訪ねてくるので「スキップ」、これで勝つる！と思ったら、そのチェックをすり抜けてまたアップロードを始めてしまう Bitcasa さん。。。例えば hoge.dat ってファイルがアップ済で、手元の hoge.data をコピーすると既に存在してる？とか訪ねてこずにいきなり hoge2.dat とかできちゃいます。。。

これは厳しい。
(Finder で Bitcasa Infinite Drive の中をよく観察していると、ファイル名がうっすらグレイで表示されているものがあります(ローカルにキャッシュされてないファイルなのかな？)。それらは更新日付が ``1984年1月24日火曜日 17:00`` になってて、これらのファイルの一致確認で「一致せず」と判断されてるのではないかと思われます。)

Terminal でも使えればスクリプトでもなんでも使えるのですが、

* [どっちが使える？：「Pogoplug cloud」vs.「Bitcasa」――容量無制限オンラインストレージ対決 (1/4) - ITmedia PC USER](http://www.itmedia.co.jp/pcuser/articles/1302/14/news019.html)

には、

> Bitcasaはコマンドプロンプトからはディレクトリが見えない（移動、作成、削除はできる）

と書いてあったのでムリかなーと思いつつ、やってみました。

```
cd /Volumes/Bitcasa\ Infinite\ Drive
ls
```

すると、、、**見える！私にも見えるぞ！**

普通にディレクトリもファイルも確認できました。``cp mv mkdir``  などのコマンドも普通に使えました。(Windows だと NG なのかな？)

これならイケるで！ということでこんなコマンドを用意。

```
cp -n -R . /Volumes/Bitcasa\ Infinite\ Drive/data
```

カレントディレクトリ(含むサブディレクトリ)のファイルを ``Bitcasa Infinite Drive/data`` ディレクトリに **存在しないファイルだけ** コピーします。

ログを取りたい場合は、

```
cp -n -R -v . /Volumes/Bitcasa\ Infinite\ Drive/data > log.txt
```

でおｋ。
注意点は ``-n`` は更新されてるかはチェックしてくれない事(というか、逆にタイムスタンプを判別したら Finder と同じ現象になると思う)

コマンドが使えるのは嬉しいですね。
Bitcasa のミラーリングバックアップはネットワークドライブに対応してませんが、定期的に cp するコマンドを走らせれば実用レベルだと思います。

## 参考
* [cp(1) OS X Manual Page](http://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man1/cp.1.html) Linux の cp コマンドと微妙に違う。。。