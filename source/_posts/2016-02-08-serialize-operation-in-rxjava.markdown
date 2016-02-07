---
layout: post
title: "RxJava で Observable の並列処理を直列化する"
date: 2016-02-08 01:16:02 +0900
comments: true
categories: [RxJava, ReactiveX, Java]
---
``rx.Observable<T>`` のオペレータは、通常は非同期で、並列に処理されます。
<!--more-->

例えば以下のような場合:

```java
public void start() {
    Observable.range(1, 5)
        .flatMap(x -> fatTask(x))
        .subscribe(x -> Log.d(TAG, "onNext - " + x));
}
 
private final Random rand = new Random();
private final ScheduledExecutorService executor = Executors.newScheduledThreadPool(5);

// ランダムにスリープした後 x を onNext する
private Observable<Integer> fatTask(final int x) {
    return Observable.create(subscriber -> {
        long sleep = (long) (rand.nextDouble() * 10000L);
        Log.d(TAG, "fatTask(" + x + ") - start.");

        executor.schedule(() -> {
            subscriber.onNext(x);
            subscriber.onCompleted();
        }, sleep, TimeUnit.MILLISECONDS);
    });
}
```

このプログラムの出力はこうなります。

> 出力:<br/>
> fatTask(1) - start.<br/>
fatTask(2) - start.<br/>
fatTask(3) - start.<br/>
fatTask(4) - start.<br/>
fatTask(5) - start.<br/>
onNext - 3<br/>
onNext - 5<br/>
onNext - 4<br/>
onNext - 2<br/>
onNext - 1<br/>

fatTask は 1,2,3,4,5 の順で *完了を待たずに* 呼びだされます。
が、それぞれ処理にかかる時間が異なるので、 ``onNext`` が呼ばれる順は 1〜 とは限りません。

ソースとなる Stream の順番を崩したくない場合は、 ``fatTask(1)`` が完了してから ``fatTask(2)`` を開始する、というように直列化しなければなりません。

## Observable.Concat(concatWith)

これを行うのが ``Observable.Concat`` です(RxJava では ``Observable.concatWith`` のようですね)。
複数の ``Observable`` を順に（完了してから次へ）処理していきます。

### 使い方

``toList`` で一旦ただの ``List`` にしてから、``concatWith`` で数珠つなぎにします。

```java
public void start() {
    Observable.range(1, 5)
        .toList()
        .flatMap(list -> {
            // fatTask(1).contat(fatTask(2)).contat(fatTask(3))... 
            // にする（fold 使えれば…)
            Observable<Integer> task = null;
            for (int x : list) {
                if (task == null) {
                    task = fatTask(x);
                } else {
                    task = task.concatWith(fatTask(x));
                }
            }
            return task;
        })
        .subscribe(x -> Log.d(TAG, "onNext - " + x));
}
```

このプログラムの出力はこうなります。

> 出力<br/>
fatTask(1) - start.<br/>
onNext - 1<br/>
fatTask(2) - start.<br/>
onNext - 2<br/>
fatTask(3) - start.<br/>
onNext - 3<br/>
fatTask(4) - start.<br/>
onNext - 4<br/>
fatTask(5) - start.<br/>
onNext - 5<br/>

``fatTask(1)`` の完了を待ってから、次の ``fatTask(2)`` が実行されています。

※
Rx.NET では、 

```csharp
static IObservable<T> Concat<T>(IEnumerable<IObservable<T>> sources)
```
 
で、複数の ``IObservable`` を一括で渡せるのですが、 RxJava にはないようで、、、。

```java
static <T> Observable<T> concatEager(Iterable<? extends Observable<? extends T>> sources)
``` 

というのがあったんですが、期待通りうごいてくれず、 Eager? なんでしょう？

## ソースが無限リストだったら？

``toList`` で一旦ただの List にしているのが非常に気に入らないですね。
``range(1, 5)`` が ``interval(1, TimeUnit.SECONDS)`` のように無限の Stream だったら使えません。

そこで、 ``concat`` には、こんな overload もあります。

```java
static <T> Observable<T> concat(Observable<? extends Observable<? extends T>> observables)
```

Observable<T> を通知する Observable？ ややこしいですがこう使います。

```java
public void start() {
    // 2. を concat する
    Observable.concat( 
        // 1. Observable<Long>
        Observable.interval(1, TimeUnit.SECONDS) 
            // 2. Long を Observable<Integer> に変換 
            //    → Observable<Observable<Integer>> になる
            .map(x -> fatTask(x.intValue()))) 
        .subscribe(x -> Log.d(TAG, "onNext - " + x));
}
```

このプログラムの出力はこうなります。

> 出力<br/>
fatTask(0) - start.<br/>
onNext - 0<br/>
fatTask(1) - start.<br/>
onNext - 1<br/>
fatTask(2) - start.<br/>
onNext - 2<br/>
…つづく<br/>

無限リストながら、並列処理せずに順序通り動いてくれます。

``interval`` の値を単純に ``map`` で ``Observable<Integer>`` に変換してやります。するとこれは ``Observable<Observable<Integer>>`` になり、``concat`` 可能になります。 ``flatMap`` だと平坦化されちゃうのでただの ``map`` です。

## まとめ

Observable は普通は非同期で並列処理。
非同期ながら直列化したい場合は ``Observable.concat`` でできます。

1. GPS から緯度経度を取得
2. なんか重い計算を行う
3. 結果をテキストファイルに書き出す

みたいな処理をするとき 3. を 1. の順序と同じにしたいのでこれを使います。

はじめ自分は ``flatMap`` で繋いでいくだけですべて直列化されているのかなーと勘違いしていたので、これを知った時は目からウロコでした。

## 参考

* [Intro to Rx - Combining sequences](http://www.introtorx.com/content/v1.0.10621.0/12_CombiningSequences.html)
* [ReactiveX - Concat operator](http://reactivex.io/documentation/operators/concat.html)
* [Reactive Extensions再入門 その４１「どんどん合成するよ」 - かずきのBlog@hatena](http://blog.okazuki.jp/entry/20120219/1329663635)
* https://twitter.com/neuecc/status/695604984763650050 - @neuecc さんありがとうございます！
