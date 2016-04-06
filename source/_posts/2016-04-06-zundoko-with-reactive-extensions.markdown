---
layout: post
title: "C# と Reactive Extensions でズンドコキヨシ"
date: 2016-03-12 23:59:59 +0900
comments: true
categories: [C#, ReactiveX, ズンドコキヨシ, .NET, ReactiveExtensions ]
---

流行り？に乗っていくスタイル。
<!--more-->

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">Javaの講義、試験が「自作関数を作り記述しなさい」って問題だったから<br>「ズン」「ドコ」のいずれかをランダムで出力し続けて「ズン」「ズン」「ズン」「ズン」「ドコ」の配列が出たら「キ・ヨ・シ！」って出力した後終了って関数作ったら満点で単位貰ってた</p>&mdash; てくも (@kumiromilk) <a href="https://twitter.com/kumiromilk/status/707437861881180160">2016年3月9日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

```csharp Zondoko.cs
var random = new Random();
var K = "キ・ヨ・シ！";
var PATTERN = new string[] { "ずん", "ずん", "ずん", "ずん", "どこ" };

Observable.Interval(TimeSpan.FromMilliseconds(100))
    .Select(_ => (random.Next() % 2 == 0) ? "ずん" : "どこ") // ランダムに ずんorどこ
    .Scan(new List<string>(), (queue, x) => // 最大５つのQueueに貯める
        {
            queue.Add(x);
            while (queue.Count > PATTERN.Count) { queue.RemoveAt(0);}
            return queue;
        })
    .SelectMany(queue => queue.SequenceEqual(PATTERN) ? // パターンと一致したら…
        Observable.Concat(
            Observable.Return(queue.Last()),   // Queueの最後
            Observable.Return(K),              // + キ・ヨ・シ！
            Observable.Return(string.Empty)) : // + 空文字(終了判定用)
        Observable.Return(queue.Last()))
    .TakeWhile(x => !string.IsNullOrEmpty(x))  // 空文字になるまで繰り返す
    .Subscribe(
        x => Console.WriteLine(x),
        () => Console.WriteLine("complete!!"));
```

>どこ<br/>
どこ<br/>
ずん<br/>
ずん<br/>
ずん<br/>
どこ<br/>
どこ<br/>
どこ<br/>
ずん<br/>
どこ<br/>
ずん<br/>
ずん<br/>
ずん<br/>
ずん<br/>
どこ<br/>
キ・ヨ・シ！<br/>
complete!!

``SelectMany`` に頼ってるのが気に入らない。。。

* RxJava 版はこちら - [RxJava でズンドコキヨシ(window 使用)](http://qiita.com/amay077/items/2c8575753e37fcc94f87)
* [さまざまなズンドコキヨシ](http://qiita.com/B73W56H84/items/519e27a1aed5e6d5304f#%E3%81%82%E3%82%8F%E3%81%9B%E3%81%A6%E8%AA%AD%E3%81%BF%E3%81%9F%E3%81%84)
* [ズンドコキヨシまとめ](http://qiita.com/shunsugai@github/items/971a15461de29563bf90)
