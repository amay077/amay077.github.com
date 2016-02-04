---
layout: post
title: "Task→Observable 変換でハマったこと"
date: 2016-01-16 23:59:59 +0900
comments: true
categories: [ReactiveX, C#]
---
.NET の ``Task<T>`` は、Reactive Extensions が提供する拡張メソッド ``ToObservable()`` で ``IObservable<T>`` に変換できます。

なにも考えずに ``ToObservable()`` を連発していたら、盛大にハマったのでメモ。
<!--more-->

## Task.Run().ToObservable() とか、意味ないっしょ

ダメなコード。

```csharp
var i = 0;
IObservable<int> incrementObservable = Task.Run (() => {
	i++;
	Debug.WriteLine($"increment! - {i}");
	return i;
})
.ToObservable ();

Debug.WriteLine("Ready...");

incrementObservable // インクリメント
	.Repeat(3) // ３回繰り返す
	.Subscribe(
		x  => Debug.WriteLine($"OnNext({x})"),
		ex => Debug.WriteLine($"OnError({ex.ToString()})"),
		() => Debug.WriteLine("OnCompleted"));

```

``incrementObservable`` は、副作用ありありですが、外部変数 i を +1 して後続に流す ``IObservable<int>`` です。
　これを ``.Repeat(3)`` して ``.Subscribe`` してますから、
　
> Ready...
> increment! - 1
> OnNext(1)
> increment! - 2
> OnNext(2)
> increment! - 3
> OnNext(3)
> OnCompleted

という出力を期待してました。
が、実際の出力はこう。

> increment! - 1
> Ready...
> OnNext(1)
> OnNext(1)
> OnNext(1)
> OnCompleted

Subscribe する前に Task が実行されてるし、 repeat してるのに increment されない。。。

「・・・ん？ Task.Run().ToObservable() って、タスクを実行した結果を IObservable 化してるだけじゃね？」

コード見たまんまなんですが、これに気づくのに１時間かかりました。。。

期待通り動くのはこう↓。

```csharp
var i = 0;
IObservable<int> incrementObservable = Observable.FromAsync(()=>Task.Run(() => {
	i++;
	Debug.WriteLine($"increment! - {i}");
	return i;
}));

Debug.WriteLine("Ready...");

incrementObservable // インクリメント
	.Repeat(3) // ３回繰り返す
	.Subscribe(
		x  => Debug.WriteLine($"OnNext({x})"),
		ex => Debug.WriteLine($"OnError({ex.ToString()})"),
		() => Debug.WriteLine("OnCompleted"));

```

``Observable.FromAsync`` で Task の実行そのものを IObservable 化します。
これの結果は正しくこう↓なりました。

> Ready...
> increment! - 1
> OnNext(1)
> increment! - 2
> OnNext(2)
> increment! - 3
> OnNext(3)
> OnCompleted


## Task は１回しか実行できない

ところで、 ``Task<T>`` は一度実行すると、２度目は実行できません。（Furure や Promise もそうだっけ）

```csharp
var i = 0;
Task<int> incrementTask = new Task<int>(() => {
	i++;
	Debug.WriteLine($"increment! - {i}");
	return i;
});

incrementTask.RunSynchronously();
incrementTask.RunSynchronously(); 
```

このコードは２回目の ``RunSynchronously()`` で例外がでます。

となると、 ``incrementTask.ToObservable()`` したとしても、期待通り動いてくれなさそうです。
（そもそも Task は ``Start`` などしないと実行されないので、Observable のチェインの中でいつ呼ぶの？）


というわけで、 ``Task.ToObservable()`` は、どういう時に使えばいいのかよくわかりませんでした。だれか教えて下さい。（汗）
