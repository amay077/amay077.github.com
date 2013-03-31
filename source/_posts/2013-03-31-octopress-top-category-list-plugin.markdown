---
layout: post
title: "Octopress Top Categories Plugin を使う"
date: 2013-03-31 16:43
comments: true
categories: [octopress]
---
サイドバーに表示するカテゴリリストを投稿数の多い順＆上位○件 の表示にするプラグインです。
<!--more-->
先生はこちら。

* [Octopress Top Categories Plugin - Time To Pull The Plug](http://time.to.pullthepl.ug/blog/2012/8/20/octopress-top-categories-plugin)
* [ctdk/octopress-category-list - GitHub](https://github.com/ctdk/octopress-category-list)


## プラグインのダウンロード
[ctdk/octopress-category-list - GitHub](https://github.com/ctdk/octopress-category-list) から ``plugins/category_list.rb`` を自分の octopress リポジトリの ``plugins/`` へコピーします。


## サイドバーに TopCategoryList を表示するための HTML を用意する
自分の octopress リポジトリの ``source/_includes/custom/asides/`` に ``top_category_list.html`` というファイルを作成、内容は以下のようにする。

```html top_category_list.html
<section>
  <h1>Top Categories</h1>
    <ul id="top-category-list">{% raw  %}{% top_category_list counter:true %}{% endraw %}</ul>
</section>
```

## top_category_list.html を使用するように設定を書き換える
自分の octopress リポジトリの ``_config.yml`` をエディタで開き、

以下のように変更する

```yml _config.yml
# [変更]custom/asides/top_category_list.html を任意の位置に挿入する
default_asides: [custom/asides/about.html, asides/recent_posts.html, custom/asides/top_category_list.html]


# [追加]カテゴリの表示件数
top_category_limit: 15
```

## 動かす

``rake generate`` して ``rake preview`` して確認した後、``rake deploy`` しましょう。


自分のブログに適用してみました。
 
* [Experiments Never Fail](http://amay077.github.com/)

Top Category List だけだと当然ながらすべてのカテゴリがわからないので、別途全てのカテゴリを列挙するページを作って、'view All-Categories' でリンクしてます。
