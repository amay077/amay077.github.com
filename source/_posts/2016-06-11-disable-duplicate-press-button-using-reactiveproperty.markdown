---
layout: post
title: "ReactiveProperty で2度押し防止(Using使ったやつ)"
date: 2016-04-07 23:59:59 +0900
comments: true
categories: [C#, ReactiveProperty, ReactiveX]
---

* [ReactivePropertyで2度押し防止 - 眠いしお腹すいたし(´・ω・`)](http://tamafuyou.hatenablog.com/entry/2016/04/06/213633)

を拝見しまして、前から気になってたので考えてみました。
<!--more-->
先に別件。

ReactiveCommand は ``IObservable<T>``、そしてロジックの方も ``IObservable<T>`` で作ることが多いのですが、その場合「ボタンをクリックした時に、ロジックを実行する」というコードは大抵以下のようになります。

```csharp MainViewModel.cs 
public class MainViewModel
{
    public ReactiveCommand TestCommand { get; }
    public ReactiveProperty<bool> IsBusy { get; } = new ReactiveProperty<bool>(false);

    public MainViewModel()
    {
        TestCommand = IsBusy.Select(x => !x).ToReactiveCommand();
        TestCommand.Subscribe(_ =>
        {
            // log("ボタンが押されたよ");
            IsBusy.Value = true;
            SomeLogicAsObservable.Subscribe(__ =>
            {
                // log("処理が実行されたよ");
                IsBusy.Value = false; // ほんとは OnCompleted と OnError でやるべき
            };
        });
    }
    
    // なんか重い非同期な処理
    private IObservable<Unit> SomeLogicAsObservable()
    {
        return Observable.Delay(Observable.Return(Unit.Default), 
                TimeSpan.FromSeconds(3));
    }    
}
```

こういうコードを書いていていつも「なんかカッコ悪いなー」と思っていました。
そう思う点は、``Subscribe`` を2回書いていること。ボタンがクリックされた事の購読の中でさらにロジックが実行された事を購読している点です。
どちらも Stream なのだから、うまくマージできないかなと思っていました。

これの解決も一緒に考えてみました。

はい、


```csharp MainViewModel.cs
public class MainViewModel
{
    public ReactiveCommand TestCommand { get; }
    public ReactiveProperty<bool> IsBusy { get; } = new ReactiveProperty<bool>(false);

    public MainViewModel()
    {
        TestCommand = TestCommand = IsBusy.Select(x => !x).ToReactiveCommand();

        TestCommand
            .SelectMany(_ => UsingIsBusy(SomeLogicAsObservable()))
            .Subscribe(_ => { /* log("ボタンが押されてロジックが実行されたよ") ※ */ });
            // ※ロジックがたくさん OnNext を呼んでいたらここもたくさん呼ばれるから注意
    }

    private IObservable<T> UsingIsBusy<T>(IObservable<T> observable)
    {
        return Observable.Using(
            () =>
            {
                IsBusy.Value = true;
                return Disposable.Create(() => IsBusy.Value = false);
            },
            _ => observable);
    }

    // なんか重い非同期な処理
    private IObservable<Unit> SomeLogicAsObservable()
    {
        return Observable.Delay(Observable.Return(Unit.Default),
                                TimeSpan.FromSeconds(3));
    }
}
```

``IsBusy`` の ON/OFF を ``Observable.Using`` に任せるようにしました。これは、第2引数に指定した ``IObservable<T>`` が完了または失敗した時にリソースを解放してくれる C# の ``using 句`` のようなもので、ここでは第2引数にロジックの ``IObservable<T>`` を渡すことで、ロジックが完了したら自動的に ``IsBusy.Value = false`` が実行されます。

次に「``Subscribe``を2回してる問題」は、ボタンがクリックされた``IObserbale<T>`` から ``SelectMany`` で繋いであげることで解決しています。

このコードの要注意は、``IsBusy.Value = false`` が実行されるのがUIスレッドとは限らないので、View側でバインドするまでに ``ObserveOnUIDispatcher`` などをしてあげないといけないことです（少なくとも Xamarin.Android + ReactiveProperty ではそうでした。WPFだと気にしないのでしょうか？）。


もう少し汎用性を高めて、「``IsBusy プロパティを持った ReactiveCommand``」のようなものを作ったら便利なのかもしれないです。
