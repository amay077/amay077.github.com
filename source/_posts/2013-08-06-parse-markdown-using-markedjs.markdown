---
layout: post
title: "Marked.js で Markdown をクライアント側でパースして表示する"
date: 2013-08-06 21:10
comments: true
categories: [Markdown, HTML5, JavaScript]
---
[Typescript クイックガイド](http://phyzkit.net/typescript/) で、ソースを見たら Markdown で書かれてたのをみて「カッケー」と思ったのと、[利用規約やプライバシーポリシーは Github に置くといいよ](http://www.atmarkit.co.jp/ait/articles/1302/05/news091.html) というのを見て、なら「github に .markdown ファイルを置いてクライアントサイドでスタイリングできたら嬉しいじゃん？」と思いやってみた。
<!--more-->
## Javascript による markdown パーサいろいろ

上記 Typescript クイックガイド では [marked.js](https://github.com/chjj/marked) が使われているようだが、他にもいろいろある模様。下記にて比較してくださっている。

* [JavaScript - Markdownパーサーのshowdown.js、markdown-js、markedを簡単比較 - ぼっち勉強会](http://kannokanno.hatenablog.com/entry/2013/06/19/132042)

もともと marked.js を使おうと思ってたし、上記サイトでも問題無さそうだったので、そのままいく。

## 使い方

marked.js を読み込んだ後、

```js 
var formatted = marked(markdown_text)
```
で、HTML に整形された文字列が返されるというシンプルなもの。(整形オプションがいろいろあるようだが割愛)

Typescript クイックガイドでは、HTML 内に Markdown テキストがベタ書きされていたが、せっかくなので管理がしやすいように外部 Markdown ファイルを読みこむようにしてみた。

```html md_renderer.html
<!DOCTYPE html>
<html lang="ja">
    <head>
        <meta charset="utf-8">

        <title>Markdown renderer</title>
        <meta name="description" content="Markdown renderer">

        <!-- bootstrap -->
        <link rel="stylesheet" type="text/css" href="css/bootstrap.css" rel="stylesheet">
        <link rel="stylesheet" type="text/css" href="css/bootstrap-responsive.css" rel="stylesheet">

        <!-- js libraries -->
        <script type="text/javascript" src="js/jquery-2.0.3.js"></script>
        <script type="text/javascript" src="js/marked.js"></script>
        <script>
            $(document).ready(function(){
                var target = $("#markdown_content");

                $.ajax({
                    url: target[0].attributes["src"].value,
                }).success(function(data){
                    target.append(marked(data));
                }).error(function(data){
                    target.append("This content failed to load.");
                });
            });
        </script>
    </head>

    <body>
        <!-- Content -->
        <div class="container">
            <div id="markdown_content" src="./sample.markdown"> </div>
        </div>
    </body>
</html>
```

一番下の ```<div id="markdown_content" src="./sample.markdown">``` の src に読み込む Markdown ファイルを指定するようにした。

bootstrap で見栄えを良くしているが、必須ではない。

こちらが [デモ](https://dl.dropboxusercontent.com/u/264530/qiita/md_renderer/md_renderer.html) 。

さすがに描画に少し時間がかかるみたいです。
``marked()`` は非同期版もあるようなので、そちらを使うべきだったかな。

これで、データ(Markdown)とプレゼンテーション部(HTML)を分離できて、なんか良さげな感じがする。

Octopress とか、HTMLを生成しちゃうんじゃなくて、クライアントサイドで 「Markdown を描画する」できると、投稿の手間が減って良いかもですね。