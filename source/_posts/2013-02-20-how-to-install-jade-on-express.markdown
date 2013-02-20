---
layout: post
title: "Express で ejs のつもりなのに jade が入っちゃう時"
date: 2012-11-20 15:17
comments: true
categories: [Node.js]
---
```
$ express -t ejs myapp
```
で ejs を入れたつもりが、```views/index.jade``` とかが入っちゃう。

```
$ express -tejs myapp
            ^スペース空けない
```
と、正しく ejs が使われる。

なんなのよこれ。express が rc だから？

##0:51追記
あーっ、ヘルプ見たら ```express -e myapp``` が正しいのね。

```
$ express -h
```

>  Usage: express [options]
>
>  Options:
>
>    -h, --help          output usage information
>    -V, --version       output the version number
>    -s, --sessions      add session support
>    -e, --ejs           add ejs engine support (defaults to jade)
>    -J, --jshtml        add jshtml engine support (defaults to jade)
>    -H, --hogan         add hogan.js engine support
>    -c, --css <engine>  add stylesheet <engine> support (less|stylus) (defaults to plain css)
>    -f, --force         force on non-empty directory

##環境
* Mac OS X 10.8.2
* node v0.9.2
* express 3.0.0rc5