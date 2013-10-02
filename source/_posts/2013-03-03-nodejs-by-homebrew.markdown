---
layout: post
title: "Homebrew で Nodes.js をやってみた"
date: 2012-10-03 20:10
comments: true
categories: [Node.js, homebrew]
---
##環境
* Mac OS X 10.7
* Homebrew はインストール済み

##参考
* [Node.jsでHello worldするまで - 飲む、寝る。](http://d.hatena.ne.jp/nomnel/20111204/1323003399)

<!-- more -->

手順はほぼこのとおり。

##手順

	$ brew install node.js

>Warning: It appears you have MacPorts or Fink installed.
Software installed with other package managers causes known problems for
Homebrew. If a formula fails to build, uninstall MacPorts/Fink and try again.
==> Downloading http://nodejs.org/dist/v0.8.4/node-v0.8.4.tar.gz
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/node/0.8.4 --without-npm
==> make install
==> Caveats
Homebrew has NOT installed npm. We recommend the following method of
installation:
  curl http://npmjs.org/install.sh | sh

>After installing, add the following path to your NODE_PATH environment
variable to have npm libraries picked up:
  /usr/local/lib/node_modules
==> Summary
/usr/local/Cellar/node/0.8.4: 79 files, 11M, built in 6.2 minutes

なんか、 MacPorts/Fink と仲が悪そうだけど、とりあえず無視して次へ。


	$ curl http://npmjs.org/install.sh | sh
>  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    85    0    85    0     0    109      0 --:--:-- --:--:-- --:--:--   384
sh: line 1: syntax error near unexpected token \`newline'
sh: line 1: `\<html>Moved: \<a href="https://npmjs.org/install.sh">https://npmjs.org/install.sh \</a>'

うお、エラーだ！どうやら URL が変わった(http → http***s*** になっただけ)らしい。
URL を修正してリトライ。

	$ curl https://npmjs.org/install.sh | sh

>  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7882  100  7882    0     0   5672      0  0:00:01  0:00:01 --:--:--  7977
tar=/usr/bin/tar
version:
bsdtar 2.8.3 - libarchive 2.8.3
install npm@1.1
fetching: http://registry.npmjs.org/npm/-/npm-1.1.62.tgz
0.8.4
1.1.62
cleanup prefix=/usr/local

>All clean!

> npm@1.1.62 prepublish .
> npm prune ; rm -rf test/*/*/node_modules ; make -j4 doc

>sh: npm: command not found
make: Nothing to be done for \`doc'.
/usr/local/bin/npm -> /usr/local/lib/node_modules/npm/bin/npm-cli.js
npm@1.1.62 /usr/local/lib/node_modules/npm
It worked
```

**npm: command not found** が気になるけど成功したぽい。

##Hello World!!

前述のサイトそのまんまなので略。
ふむ、うまくいった。

