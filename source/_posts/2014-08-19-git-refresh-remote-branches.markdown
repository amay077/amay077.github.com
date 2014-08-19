---
layout: post
title: "Git でリモートリポジトリの更新が反映されないとき"
date: 2014-08-19 23:52:21 +0900
comments: true
categories: [git, SourceTree]
---
GitHub のWebサイトでブランチを削除したあと、クライアント（SourceTreeとか）のリモートブランチの表示に、削除したはずのブランチが残っていて、気持ち悪いなあ、と思っていた。
<!--more-->

git のコマンド一発だった。

```sh
git remote update -p
```

これでクライアント側のリモート情報がリフレッシュされる。