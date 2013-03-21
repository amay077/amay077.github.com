---
layout: post
title: "AsyncTask の cancel メソッドにはバグがあるよ、Froyo(2.2)以降で直されるかもよ→Gingerbread（2.3）で直ったよ"
date: 2010-11-16 9:01
comments: true
categories: [Android, Java] 
---
### 2010.12.16 追記

Gingerbread (Android 2.3) では修正されているとの事です。
<!--more-->

>Romain Guy:
> This AyncTask's cancel bug was fixed in Gingerbread?
> It was fixed in Gingerbread yes :)
> via [groups.google.com - AyncTask's cancel bug was fixed in Gingerbread?](http://groups.google.com/group/android-developers/browse_thread/thread/6dccc5cbd7bb9205/aae9a3136945890c#)

2.3 のコードが公開されれば、2.2 でも ”正しく” 修正できますね。

- 追記ここまで -

[AsyncTaskの cancel()には問題があり、Froyoの次で修正される。
- kinneko@転職先募集中の日記](http://d.hatena.ne.jp/kinneko/20100730/p9)

で教えてもらった話。

> Eric Mill:
> 1) onDestroy(), calls task.cancel() 
> 
> 2) task's onPostExecute runs, isCancelled() returns false, so I have 
> no conditional to stop the flow
> 
> and that's it. onCancelled never runs.
> 
> via [groups.google.com](http://groups.google.com/group/android-developers/browse_thread/thread/07ea01892ee7a5f4/9f71428217c2cd44?)

Activity.onDestroy で AsyncTask.cancel を呼ぶと、

* キャンセル時は呼ばれないハズの onPostExecute が呼ばれちゃう。
* isCancelled は false を返すし、
* onCancelled も呼ばれないし、

から、なんとかしてぇ！ということらしい。

んで Google の Romain さんが、

> Romain Guy:
> There was a race in the cancel() code that we fixed post-froyo. 

と言ってるけど、"post-froyo" というのは、
Post-froyo  means after froyo, so not in froyo :) 
だそうで、”Froyo の後” つまり、Gingerbread (Android 2.3) での修正らしい。
（けど "gingerbread!"  って断言してないからもっと先かも知れないなあ。）
なので、しばらくは、いや当分は自力で flag でも用意して回避するしかないみたい。
けど AsyncTask.cancel は final メソッドで override できないから、何とも不自然なコードになりそう。。。 