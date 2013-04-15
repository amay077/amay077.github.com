---
layout: post
title: "Bitcasa(Mac)が不安定になったらこのコマンドを叩け"
date: 2013-04-12 14:18
comments: true
categories: [Bitcasa]
---
[Bitcasa を使っていて「おや？」と思ったこと](http://amay077.github.com/blog/2013/03/25/bitcasa-tips-1/) や [だんだんと Bitcasa が嫌になってきたでござるよ](http://amay077.github.com/blog/2013/04/01/bitcasa-troubles-1/) でさまざまな Bitcasa の問題と対峙してきたわけですが、やっとそれらを克服する方法がわかりました。
<!--more-->
問題が起こったら、アプリを終了させて下のコマンドを叩く！

```sh Bitcasa_clear_cache.sh
rm -r ~/Library/Caches/com.bitcasa.Bitcasa
rm -r ~/Library/Application\ Support/com.bitcasa.Bitcasa 
```

いやー、Caches の方は知ってて何度もトライしてたんですけど、Application Support の方は知りませんでしたわ。(むしろ Caches は Application Support のシンボリックリンクで、削除しても無意味というウワサも…)

ともあれ、この操作を行った後は、ファイルのアップロードを延々と繰り返すこともなくなった※し、Finder でコピー時にエラーが発生することもなくなり、安定した気がします。

※嘘でした。程なくして2GBのアップロードを繰り返すようになりました(泣)

出典はコチラ

* [Bitcasa is hanging in status "Finalizing" : Bitcasa Help Center](http://support.bitcasa.com/entries/22943756-Bitcasa-is-hanging-in-status-Finalizing-)

で、Kyle さんが @Jon さんに回答してるトコです

>@Jon: The "Finalizing" message usually indicates  a hiccup in the cache. We're working to figure out the root cause of this, and expect a fix in an upcoming release. In the meantime, you can work around it by doing the following:
>
> 1 Exit the Bitcasa Desktop App
>
> 2 Open Terminal and type:
> 
> rm -r ~/Library/Caches/com.bitcasa.Bitcasa
>
> 3 Press Enter, and wait few minutes
> 
> 4 Next, type:
>
> rm -r ~/Library/Application\ Support/>com.bitcasa.Bitcasa 
>
> 5 Relaunch the Bitcasa Desktop App
>
>April 04, 2013 03:04 pm 