---
layout: post
title: "Hot Observable と ConnectableObservable について"
date: 2015-12-17 23:59:59 +0900
comments: true
categories: [RxJava, Android, Java, ReactiveX]
---
　これは [RxJava Advent Calendar 2015 17日目](http://qiita.com/advent-calendar/2015/rxjava) の記事です。

空いてたので参加してみました。
普段は Xamarin(C#) + Reactive Extensions + ReactiveProperty で、Reactive + MVVM な Android/iOS両対応アプリを開発しています。
<!--more-->

## Cold vs Hot

Cold Observable は「あなただけの」Stream、Hot は「みんなの」Stream 。
（私的にはニコ動かニコ生か、みたいに理解してますが、その話はいいや）

Cold は、あなたが subscribe した瞬間からデータが流れ始めます。
Hot は、あなたが subscribe してもデータは流れ始めません(流れるかも知れません？)。

では Hot Observable はいつからデータが流れ始める？Observable が生成された瞬間から？
その答え(の一つ)が **ConnectableObservable** 。

## ConnectableObservable のデータ放流の開始と停止

Cold Observable を Hot化する publish メソッドの返り値は ConnectableObservable。
Hot は必ず ConnectableObservable。（←これ後で否定します）

ConnectableObservable には connect メソッドがあります。
Hot Observable のデータが流れ始めるのは、このメソッドを呼んだ瞬間から。
なので、どれだけ subscriber が居ようとも connect を呼ばなければデータは流れません。逆に subscriber が居なくても connect を呼べばデータが流れ始めます。

connect メソッドの返り値は Subscription です。
Subscription の unsubscribe メソッドを呼ぶと、データの放流が停止します。これも subscriber が居ようが居まいが停止します。
再度 connect すると、 **最初から** データが流れ出します。再開ではありません。

## 実例

### Cold Observable

Observable.interval は、一定時間置きにインクリメントされた値を流す **Cold** Observable。
なので、複数の subscriber が居たら、各々に独立した値を流します。

Android の画面にボタンが２つ（buttonSubscribe1 と buttonSubscribe2）並んでるだけのサンプルです。

```java
final Observable<Long> tickObservable = Observable.interval(1000, TimeUnit.MILLISECONDS);

// 可視性向上の為のなんちゃってラムダ
findViewById(R.id.buttonSubscribe1).setOnClickListener(v -> { 
    Log.d(TAG, "buttonSubscribe1 click!");
    tickObservable.subscribe(x -> Log.d(TAG, "subscriber1 - onNext - " + x));
});

findViewById(R.id.buttonSubscribe2).setOnClickListener(v -> {
    Log.d(TAG, "buttonSubscribe2 click!");
    tickObservable.subscribe(x -> Log.d(TAG, "subscriber2 - onNext - " + x));
});
```

>結果:<br/>
D/MainActivity: buttonSubscribe1 click!<br/>
D/MainActivity: subscriber1 - onNext - 0<br/>
D/MainActivity: subscriber1 - onNext - 1<br/>
D/MainActivity: subscriber1 - onNext - 2<br/>
D/MainActivity: subscriber1 - onNext - 3<br/>
D/MainActivity: subscriber1 - onNext - 4<br/>
D/MainActivity: subscriber1 - onNext - 5<br/>
D/MainActivity: buttonSubscribe2 click!<br/>
D/MainActivity: subscriber1 - onNext - 6<br/>
D/MainActivity: subscriber2 - onNext - 0<br/>
D/MainActivity: subscriber1 - onNext - 7<br/>
D/MainActivity: subscriber2 - onNext - 1<br/>
D/MainActivity: subscriber1 - onNext - 8<br/>
D/MainActivity: subscriber2 - onNext - 2<br/>

buttonSubscribe1 を押すとデータ(０から連番)が流れ始めます。
しばらくして buttonSubscribe2 を押すと、1 とは関係なく、また 0 から流れ始めます。

### Hot(Connectable) Observable

publish で Hot 化します。
connect と unsubscribe を呼ぶためのボタン（buttonConnect, buttonDisConnect）を画面に追加してます。

```java
private Subscription _connection; // field です


final ConnectableObservable<Long> tickObservable = 
    Observable.interval(1000, TimeUnit.MILLISECONDS).publish(); // publish で Hot化

findViewById(R.id.buttonSubscribe1).setOnClickListener(v -> { 
    Log.d(TAG, "buttonSubscribe1 click!");
    tickObservable.subscribe(x -> Log.d(TAG, "subscriber1 - onNext - " + x));
});

findViewById(R.id.buttonSubscribe2).setOnClickListener(v -> {
    Log.d(TAG, "buttonSubscribe2 click!");
    tickObservable.subscribe(x -> Log.d(TAG, "subscriber2 - onNext - " + x));
});

findViewById(R.id.buttonConnect).setOnClickListener(v -> {
    Log.d(TAG, "buttonConnect click!");
    _connection = tickObservable.connect(); // データ放流開始
});

findViewById(R.id.buttonDisConnect).setOnClickListener(v -> {
    Log.d(TAG, "buttonDisConnect click!");
    if (_connection != null) {
        _connection.unsubscribe(); // データ放流停止
        _connection = null;
    }
});
```

>結果:<br/>
D/MainActivity: buttonSubscribe1 click!<br/>
D/MainActivity: buttonConnect click!      // ←数秒経過している<br/>
D/MainActivity: subscriber1 - onNext - 0<br/>
D/MainActivity: subscriber1 - onNext - 1<br/>
D/MainActivity: subscriber1 - onNext - 2<br/>
D/MainActivity: subscriber1 - onNext - 3<br/>
D/MainActivity: buttonSubscribe2 click!<br/>
D/MainActivity: subscriber1 - onNext - 4<br/>
D/MainActivity: subscriber2 - onNext - 4<br/>
D/MainActivity: subscriber1 - onNext - 5<br/>
D/MainActivity: subscriber2 - onNext - 5<br/>
D/MainActivity: subscriber1 - onNext - 6<br/>
D/MainActivity: subscriber2 - onNext - 6<br/>
D/MainActivity: buttonDisConnect click!<br/>
-これ以降 onNext は出力されない-<br/>

buttonSubscribe1 を押しても、まだデータは流れてきません。
数秒後、buttonConnect を押すとデータが流れ始めます。
buttonSubscribe2 を押すと、subscriber2 が増えますが、Hot(みんなの)Observable なので、流れてくる値とタイミングは subscriber1 と全く同じです。

buttonDisConnect を押すと、データの放流が停止されます。(ちなみにもう一度 CONNECT すると、また 0 から値が流れます)
subscriber1, subscriber2 にはもう onNext は呼ばれません。

※サンプルでは onNext しか受信していませんが、 buttonDisConnect を押しても、 subscriber1, subscriber2 の onComplete や onError も呼ばれません。つまり、 **「データの放流が停止されても、 subscriber はそれに気付けない」** ということになります。これはこれでいいんだろか、という感じです。

## ConnectableObservable.refCount について

> Hot は必ず ConnectableObservable。（←これ後で否定します）

否定始めます。

ConnectableObservable では、データ放流の開始と停止は、 connect と unsubscribe に委ねられていました。

refCount() を使うとそれを自動化できます。(refCount？参照カウントを返すメソッド？そう思っていましたが全然違いました。)
どういうことかと言うと、最初の subscriber が現れたらデータ放流を開始し、誰も subscriber が居なくなったら放流を停止する、というものです。
refCount() の返値はただの Observable です、でも Hot です。はい否定しましたー。

### 実例

publish した Hot Observable を refCount してデータ放流を自動制御してもらいます。
画面には、 buttonConnect, buttonDisConnect に代わり、buttonUnsubscribe1, buttonUnsubscribe2 を用意します。

```java
private Subscription _subscription1; // field です
private Subscription _subscription2; // field です
private Subscription _connection;    // field です


final Observable<Long> tickObservable = 
    Observable.interval(1000, TimeUnit.MILLISECONDS).publish().refCount(); // 返値は Connectable ではない

findViewById(R.id.buttonSubscribe1).setOnClickListener(v -> { 
    Log.d(TAG, "buttonSubscribe1 click!");
    _subscription1 = tickObservable.subscribe(x -> Log.d(TAG, "subscriber1 - onNext - " + x));
});

findViewById(R.id.buttonSubscribe2).setOnClickListener(v -> {
    Log.d(TAG, "buttonSubscribe2 click!");
    _subscription2 = tickObservable.subscribe(x -> Log.d(TAG, "subscriber2 - onNext - " + x));
});

findViewById(R.id.buttonUnsubscribe1).setOnClickListener(v -> {
    Log.d(TAG, "buttonUnsubscribe1 click!");
    if (_subscription1 != null) {
        _subscription1.unsubscribe(); // 1購読終了
        _subscription1 = null;
    }
});

findViewById(R.id.buttonUnsubscribe2).setOnClickListener(v -> {
    Log.d(TAG, "buttonUnsubscribe2 click!");
    if (_subscription2 != null) {
        _subscription2.unsubscribe(); // 2購読終了
        _subscription2 = null;
    }
});
```

>結果:<br/>
D/MainActivity: buttonSubscribe1 click!<br/>
D/MainActivity: subscriber1 - onNext - 0<br/>
D/MainActivity: subscriber1 - onNext - 1<br/>
D/MainActivity: subscriber1 - onNext - 2<br/>
D/MainActivity: buttonSubscribe2 click!<br/>
D/MainActivity: subscriber1 - onNext - 3<br/>
D/MainActivity: subscriber2 - onNext - 3<br/>
D/MainActivity: subscriber1 - onNext - 4<br/>
D/MainActivity: subscriber2 - onNext - 4<br/>
D/MainActivity: subscriber1 - onNext - 5<br/>
D/MainActivity: subscriber2 - onNext - 5<br/>
D/MainActivity: subscriber1 - onNext - 6<br/>
D/MainActivity: subscriber2 - onNext - 6<br/>
D/MainActivity: subscriber1 - onNext - 7<br/>
D/MainActivity: subscriber2 - onNext - 7<br/>
D/MainActivity: buttonUnsubscribe1 click!<br/>
D/MainActivity: subscriber2 - onNext - 8<br/>
D/MainActivity: subscriber2 - onNext - 9<br/>
D/MainActivity: subscriber2 - onNext - 10<br/>
D/MainActivity: subscriber2 - onNext - 11<br/>
D/MainActivity: buttonUnsubscribe2 click!<br/>
-これ以降 onNext は出力されない-<br/>

buttonSubscribe1 を押すと、その時点でデータが流れ始めます(refCount による自動制御)。
buttonSubscribe2 を押すと、subscriber1 と同じタイミングで、同じ値を受信できます(Hot だから)。
buttonUnsubscribe1 を押すと、 subscriber1 は購読をやめますが、subscriber2 はまだ受信しています。
buttonUnsubscribe2 を押すと、subscriber2 も購読をやめ、この時点でデータ放流が停止します(refCount による自動制御)。

※ほんとにデータ放流終わってんの？を確認するには、 tickObservable に doOnNext を繋げて確認するとよいと思います。

## まとめ

Hot Observable は、ほとんどの場合(publish により生成されるので) ConnectableObservable。
ConnectableObservable は、購読者の有無に関係なく connect でデータ放流開始、Subscription.unsubscribe でデータ放流停止。
refCount により購読者の有無に連動したデータ放流の自動制御が可能。この場合 Hot だけど普通の Observable型。

実際に Hot Observable を使う場合は、refCount() しとくのが無難かなー、と思いました。(購読者の unsubscribe を厳密に管理しておけば、という前提で)


## 参考

* [Intro to Rx - Hot and Cold observables](http://www.introtorx.com/content/v1.0.10621.0/14_HotAndColdObservables.html)
* [Connectable Observable Operators · ReactiveX/RxJava Wiki](https://github.com/ReactiveX/RxJava/wiki/Connectable-Observable-Operators)
* [RxJava Advent Calendar 2015 を書かれた皆さん](http://qiita.com/advent-calendar/2015/rxjava)
