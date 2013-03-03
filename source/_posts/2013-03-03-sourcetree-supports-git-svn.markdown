---
layout: post
title: "SourceTree は git-svn したリポジトリも使えます"
date: 2012-10-03 16:19
comments: true
categories: [Mac, Subversion, git, SourceTree]
---
Mac 最強の git/Mercurial クライアントである [SourceTree](http://www.sourcetreeapp.com/) ですが、git-svn した Subversion リポジトリもほとんどシームレス い扱うことができます。
<!--more-->
以下、方法です。

I. Terminal で git svn clone する。※リポジトリがデカイと結構時間かかります。

```sh
git svn clone -s http://osmdroid.googlecode.com/svn osmdroid
```
-s オプションをつけているので、最後 /trunk としない所がミソ。
 
II. SourceTree の ファイル → 開く で I. のフォルダを作業フォルダとして開く。

III. ごらんの通り、普通に SourceTree で開けました。

![sourcetree](https://dl.dropbox.com/u/264530/qiita/sourcetree01.png)

* プッシュは ``git svn dcommit`` と同じ、つまり SVN へのコミットになります。
* プルは ``git svn rebase`` つまり SVN からの Update になります。
* ブランチは少し注意が必要で、git のローカルブランチとして作成するか、SVN のブランチとして作成するかを選択するダイアログが表示されます。前者を選択した場合は、SVN へは手動でマージしなければなりません。
![sourcetree_branch](https://dl.dropbox.com/u/264530/qiita/sourcetree02.png)

git svn のコマンドは普通の git と少し違うのですが、SourceTree がそのあたりを吸収してくれて、SVN をリモートリポジトリと見立てて動作します。

あわよくば、SVNリポジトリの URL を直接 SourceTree で開けると尚便利でしたが、まあいいでしょう。

SVN の不便なところに気軽にローカルブランチが作れないというのがあったので、git-svn を使うことでローカルブランチ作り放題です。

Mac の SVNクライアントで「これだ！」というのがなかったのですが、これで満足です。

##参考
* [git-svnの使い方を覚えた - idesaku blog](http://d.hatena.ne.jp/idesaku/20090323/1237825080)
* [SourceTree](http://www.sourcetreeapp.com/) - なんか "Supports SVN via git-svn or hgsubversion" って書いてあるので git-svn の代わりに hgsubversion でもいけそうですね。