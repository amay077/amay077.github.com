---
layout: post
title: "Rx で Chain of Responsibility"
date: 2014-05-13 15:24
comments: true
categories: [ReactiveExtensions, C#]
---
今さらだけど GoF の Chain of Responsibility パターン。「自分に処理できないタスクは上へ投げる」ってやつ。Reactive な感じでやるとこんな感じかなあと思って書いてみた。
<!--more-->

```csharp
void Main()
{
    var document = "有給届";
    
    var kakariCho = CreateManager("係長", document, d => String.Equals(document, "遅刻届"));
    var kaCho = CreateManager("課長", document, d => String.Equals(document, "有給届"));
    var buCho = CreateManager("部長", document, d => String.Equals(document, "退職届"));
    
    Observable.Concat(new [] { 
        kakariCho,  // 係長
        kaCho,      // 課長
        buCho       // 部長
    })
    .FirstOrDefault() // 最初の１人に承認されたら終了
    .Timeout(TimeSpan.FromDays(1)) // 猶予１日
    .Subscribe(x => Debug.WriteLine(String.IsNullOrEmpty(x) 
        ? "あなたの届書は却下されました" 
        : x + "が承認しました"));
}

/// 管理職を作成する（役職名、渡された届書、自分に承認できる届書）
IObservable<string> CreateManager(string managerTitle, string document, Predicate<string> canIAccept)
{
    return Observable.Create<string>(o => Task.Run(() => 
    {
        if (canIAccept(document)) 
        {
            o.OnNext(managerTitle); // 承認
        }
        o.OnCompleted();
    }));
}
```

管理職の人を ``IObservable`` に見立てて、自分が処理できるなら ``OnNext`` を呼ぶ、処理できないなら ``OnNext`` は呼ばずに ``OnComplete`` しちゃう。
で、係長・課長・部長の IObservable を ``Concat`` で役職の低い順につなげて、 ``FirstOrDefault()`` で最初の承認がもらえるまで待つ、みたいな。

係長・課長・部長が誰も承認しなかった時、タイムアウトするまで待ちが発生しちゃうのが難点。→ ``Take(1)`` じゃなくて ``FirstOrDefault`` すればいいみたい。誰も承認しなかった場合 ``default(string)`` つまり空文字が流れてくる。

あと、係長→課長→部長と管理職のハンコリレーが必要な場合に対応できていない、Concat なので係長の結果を課長に引き継いでないから。

んーどうしようか。。。
