---
layout: post
title: "RxJava の ImmediateScheduler と TrampolineScheduler の違い"
date: 2016-02-05 01:30:59 +0900
comments: true
categories: [ReactiveExtensions, RxJava, ReactiveX]
---
RxJava のスケジューラの中に [``TrampolineScheduler``](http://reactivex.io/RxJava/javadoc/rx/schedulers/TrampolineScheduler.html) というのがあり、[なんじゃこれ？](https://twitter.com/amay077/status/693341525464346624)とつぶやいたところ、 [Rx.NET の ``CurrentThreadScheduler`` と同じっぽい](http://reactivex.io/RxJava/javadoc/rx/schedulers/TrampolineScheduler.html) と教えてもらいました。
<!--more-->

その流れで、類似の Scheduler である [ImmediateScheduler](http://reactivex.io/RxJava/javadoc/rx/schedulers/ImmediateScheduler.html) との違いについて語られているトピックを紹介してもらいました。

<blockquote class="twitter-tweet" lang="ja"><p lang="ja" dir="ltr"><a href="https://twitter.com/amay077">@amay077</a> この辺読みとくと良いと思います（tranpolineというキーワードも登場します） <a href="https://t.co/A5TzOiobsC">https://t.co/A5TzOiobsC</a></p>&mdash; Atsushi Eno (@atsushieno) <a href="https://twitter.com/atsushieno/status/693396949643317248">2016, 1月 30</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

# CurrentThreadScheduler vs ImmediateScheduler

* [CurrentThreadScheduler vs ImmediateScheduler](https://social.msdn.microsoft.com/Forums/en-US/7f75482f-eff2-4938-9491-47fe870989e8/currentthreadscheduler-vs-immediatescheduler?forum=rx)

紹介してもらったこのディスカッションを、頑張って翻訳してみました（めちゃくちゃなとこは訂正願います）。
RxJava では ``CurrentThreadScheduler`` を ``TrampolineScheduler`` に読み替えてください。

－－訳ここから－－

## Ohad 氏の質問

> Hi
> ImmediateScheduler's Schedule method is pretty straightforward - it simply invokes the action.
In contrast, CurrentThreadScheduler seems more involved - it creates something called a trampoline, which in turn iterates over an action queue, sleeping between invocations of items in the queue and so forth

やあ、
``ImmediateScheduler.Schedule`` メソッドは単純をアクションを呼び出します。
対照的に、 ``CurrentThreadScheduler`` は複雑に見えます。トランポリンと呼ばれるものを作り、それはアクション・キューで、順次スリープの間に呼び出されます。


> I've been trying to follow the code with reflector but I'm having a hard time understanding the difference. As far as I can tell, CurrentThreadSchedule's schedule method calls Trampoline 's Run method, which will end up blocking the current thread until the queued action is performed (on the current thread as well) - apparently just like in the case of ImmediateScheduler

私はコードを追ってみましたが、理解するのに苦労しています。分かる範囲では、 ``CurrentThreadSchedule.Schedule`` メソッドは ``Trampoline.Run`` メソッドを呼び出しています。これは現在のスレッドを、キューのアクションが実行されるまで(カレントスレッドも同様に)ブロックしようとします。 - どうも ``ImmediateScheduler`` のようにみえます。


> I realize I'm missing something, so an explanation would be really appreciated
>Thanks !

何か理解が足りないと思うので、説明してもらえると嬉しいです。


> EDIT - In the meantime I've found a couple of resources that may shed light on the subject, if anyone's interested:

２つのリソースを見つけました。何かの手がかりになれば。

http://channel9.msdn.com/blogs/j.van.gogh/controlling-concurrency-in-rx

http://community.bartdesmet.net/blogs/bart/archive/2009/11/08/jumping-the-trampoline-in-c-stack-friendly-recursion.aspx

## Dave 氏の回答

> Hi,
> 
> The trampoline seems to serve three purposes: 

やあ、
トランポリンは３つの目的を持っているように見えます。

> 1- Prevents dead-locks from scheduler reentrancy.

1- スケジューラーの割り込みからデッドロックを防ぎます。

> 2- Prevents infinite loops in observables that require recursion through scheduler reentrancy.

2- スケジューラーの割り込みを使った再帰が必要な Observable の無限ループを防ぎます。

> 3- Cooperative single-threaded multitasking; I guess it's similar to the proposed async/await feature in C# 5.0.  Calling CurrentThreadScheduler.Schedule is sort of like using await when the currently executing code was also scheduled via CurrentThreadScheduler.

3- シングルスレッドでの「[協調的マルチタスキング](http://www.sophia-it.com/content/%E3%83%8E%E3%83%B3%E3%83%97%E3%83%AA%E3%82%A8%E3%83%B3%E3%83%97%E3%83%86%E3%82%A3%E3%83%96%E3%83%9E%E3%83%AB%E3%83%81%E3%82%BF%E3%82%B9%E3%82%AF)」; 私は C# 5.0 に提案されている async/await に近いものだと思います。``CurrentThreadScheduler.Schedule`` の呼び出しは、現在実行中のコードも ``CurrentThreadScheduler`` でスケジュールされていたときに await を使用するようなものです。（訳注: C# の async/await は協調的マルチタスキングではないと思います。これは async/await 登場以前に予想で書かれたものかと。

> In the observable world, calling Subscribe should be an asynchronous operation.  There's a problem if the scheduling of an observable dead-locks or blocks the current thread indefinitely because it attempts to execute immediately and never completes.

Observable の世界では、``Subscribe`` の呼び出しは、非同期処理で行わなければなりません。Observable のスケジューリングがデッドロックまたはカレントスレッドを無期限にブロックする場合、すぐに実行しようとしても完了しないので、問題になります。

> Ignore the type of scheduler for a moment and consider a scheduled action that eventually, through some sequence of method calls, uses the same scheduler to schedule another action.

ちょっとこのスケジューラを無視して、いずれは、いくつかのシーケンスは別のアクションをスケジュールするために、同じスケジューラを使用することを考えてみてください。

> With the ImmediateScheduler, the inner action is executed immediately.

``ImmediateScheduler`` では、”内側のアクション” はすぐに実行されます。

> * If the outer action acquires some resource on which the inner action depends, and the inner action cannot acquire this resource until it's released by the outer action, then these actions dead-lock.

* 外側のアクションが、内側のアクションが依存しているリソースを取得した場合、
内側のアクションは外側のアクションがリソースを開放するまでそれを取得できず、これらのアクションはデッドロックします。

> * If the outer action depends upon the inner action, and the inner action depends upon the outer action, then this could result in an infinite loop that never yields control to other actions.

* 外側のアクションは内部アクションに依存し、内部アクションは外側の行動に依存している場合、他のアクションに制御が移らない無限ループになります。

> For example: Observable.Return(1).Repeat().Take(1)

例: ``Observable.Return(1).Repeat().Take(1)``

> By default, Return uses the ImmediateScheduler to call OnNext(1) then OnCompleted().  Repeat does not introduce any concurrency, so it sees OnCompleted immediately and then immediately resubscribes to Return.  Because there's no trampoline in Return, this pattern repeats itself, blocking the current thread indefinitely.  Calling Subscribe on this observable never returns.  See [this discussion](https://social.msdn.microsoft.com/Forums/en-US/f9c1a7a6-d6a3-44fd-ba8c-e6845b1717b2/possible-bug-repeat-observables-using-immediate-scheduler?forum=rx) for more information.

既定では、``Return`` は ``ImmediateScheduler`` を使って ``OnNext(1)`` そして ``OnCompleted()`` を呼び出します。 ``Repeat`` はどんな並列性も使用しません、なのですぐに ``OnCompleted`` を検知して、すぐに ``Return`` を再購読します。なぜなら、 ``Return`` にはトランポリンがないので、このパターンは自分自身を繰り返し、無期限に現在のスレッドをブロックし続けます。この Observable を ``Subscribe`` すると処理が返ってきません。詳細については、[この説明](https://social.msdn.microsoft.com/Forums/en-US/f9c1a7a6-d6a3-44fd-ba8c-e6845b1717b2/possible-bug-repeat-observables-using-immediate-scheduler?forum=rx)を参照してください。

> With the CurrentThreadScheduler, the inner action is scheduled (queued) for execution when the outer action ends.  Conceptually, inner actions are bounced on the trampoline until the current thread is ready to execute them.

``CurrentThreadScheduler`` では、内側のアクションは、外側のアクションが終了された時に実行されるようにスケジュールされます。コンセプトとしては、内側のアクションは、現在のスレッドが実行可能になるまでトランポリンの上で跳ねます。

> * If the outer action acquires some resource on which the inner action depends, and the inner action cannot acquire this resource until it's released by the outer action, then these actions do not dead-lock because the inner action is not executed until the outer action completes.

* 外側のアクションが、内側のアクションが依存しているリソースを取得し、内側のアクションは外側のアクションによってそれらが解放されるまで取得できない場合、これらのアクションはデッドロックしません、なぜなら。内側のアクションは外側のアクションが終了するまで実行されないためです。

> * If the outer action recurses when the inner action completes, then there won't be an immediately infinite loop because the inner action does not complete until the outer action completes first.

* 外側のアクションが内側のアクションが終了した時に再帰的な場合、無限ループになりません。なぜなら、内側のアクションは外側のアクションが完了するまで完了しないためです。

> For example: Observable.Return(1, Scheduler.CurrentThread).Repeat().Take(1)

例: ``Observable.Return(1, Scheduler.CurrentThread).Repeat().Take(1)``

> Here, Return is using the CurrentTheadScheduler to call OnNext(1) then OnCompleted().  Repeat does not introduce any concurrency, so it sees OnCompleted immediately and then immediately resubscribes to Return; however, this second subscription to Return schedules its (inner) actions on the trampoline because it's still executing on the OnCompleted callback from the first scheduled (outer) action, thus the repetition does not occur immediately.  This allows Repeat to return a disposable to Take, which eventually calls OnCompleted, cancels the repetition by disposing Repeat, and ultimately the call from Subscribe returns.

ここでは、 ``Return`` は ``CurrentTheadScheduler`` を使って ``OnNext(1)`` そして ``OnCompleted()`` を呼び出します。 ``Repeat`` はどんな並列性も使用しません、なのですぐに ``OnCompleted`` を検知して、すぐに ``Return`` を再購読します。しかし、この２回目の ``Return`` の購読（内側のアクション）はトランポリンの上にあります、なぜなら、最初にスケジュールされたアクション（外側のアクション）の ``OnCompleted`` コールバックの上でまだ実行中であるからです、なので繰り返しはすぐに発生しません。
これは、``Repeat`` は ``Take`` に disposable(subscription) を返すことができます、それはやがて ``OnCompleted`` を呼び出し、``Subscribe`` の返値から ``Repeat`` の破棄により繰り返しをキャンセルします。

> Keep in mind that the examples with Return and Repeat do not introduce any concurrency.  When you call Subscribe, it will not return until the observable completes regardless of which of these schedulers you choose.  With the ImmediateScheduler, Take calls OnCompleted but it cannot cancel the repetition, so Subscribe blocks indefinitely.  Alternatively, the CurrentThreadScheduler allows for cooperative single-threaded multitasking between the Return and Repeat operators, thus allowing Take to cancel the repetition without having to introduce any concurrency.

覚えておいて欲しいのは、 ``Return`` → ``Repeat`` はどんな並列性も使用しないことです。``Subscribe`` を呼び出すと、あなたが選択した Scheduler に関係なく、Observable が終了するまで処理を返しません。 ``ImmediateScheduler`` では、``Take`` は ``OnCompleted`` を呼び出しますが、繰り返しをキャンセルできません。なので ``Subscribe`` は無期限にブロックしてしまいます。代わりに ``CurrentThreadScheduler`` は ``Return`` と ``Repeat`` 操作の間、協調的マルチタスキングが可能になります、なので、並行性を使用することなく、繰り返しをキャンセルすることができます。

－－訳ここまで－－

# つまり？

``ImmediateScheduler`` が処理をただ単に(割り込んで)実行するだけであるのに対し、 ``CurrentScheduler``(``TrampolineScheduler``) は、擬似的なマルチタスクを行う（懐かしの VB の [``DoEvents``](http://detail.chiebukuro.yahoo.co.jp/qa/question_detail/q1112681621) かぁ？）ことでデッドロックを防いでいる、と理解しました。

Dave 氏の回答にあった例

```csharp
Observable.Return(1).Repeat().Take(1).Subscribe(...);
Debug.WriteLine("Hoge");
```

を実行すると、確かに処理が帰ってこない、``Debug.WriteLine`` へ進まないんです。

これはヤバい、``ImmediateScheduler`` マジやべえと。
で、 RxJava でも同じだよねえと、

```java
// just が ImmediateScheduler 使うのか不安だったから subscribeOn しているよ
Observable.just(1).subscribeOn(Schedulers.immediate()).repeat().take(1).subscribe(...);
Log.debug(TAG, "Hoge");
```

と書いて実行してみたら、処理が帰ってくる！ ``Log.debug`` も実行される！
なんだこの違いは？改善されているのか、試し方が悪いのか。。。

なんだかモヤモヤした終わりかたですが、今回は ``ImmediateScheduler`` はちょっと要注意だというところまでです。

> In the observable world, calling Subscribe should be an asynchronous operation.

との言葉通り、Rx.NET/RxJava を使うときは非同期にしたい事が全てだと思いますが、オペレータによっては既定で ``ImmediateScheduler`` を使うものもあるので、必ず ``subscribeOn/observeOn`` をするクセをつけておいた方がいいのかな？と思いました。
