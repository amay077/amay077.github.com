---
layout: post
title: "RxJava の Observable を Android DataBinding の ObservableField に変換する"
date: 2016-02-27 01:19:03 +0900
comments: true
categories: [Android,RxJava]
---

　Android DataBinding で View とバインドできるデータクラスは ``BaseObservable`` から派生したクラスか、``ObservableField<T>`` 型のフィールドのみです。
<!--more-->

　RxJavaベースの API やモデルクラスを使用している場合、更新通知は ``rx.Observable<T>`` を ``subscribe`` することで受けられるわけですが、それを View にバインドするには、``ObservableField<T>`` に変換してあげなければなりません。

　結果、下のような Utility 関数を作ることになります。

```java
/**
 * rx.Observable から ObservableField への変換をおこなう
 */
static public <T> ObservableField<T> toObservableField(Observable<T> source, CompositeSubscription subscriptions) {
    final ObservableField<T> field = new ObservableField<T>();

    subscriptions.add(
            // TODO onError も拾ったほうがいい
            source.subscribe(new Action1<T>() {
                @Override
                public void call(T x) {
                    field.set(x);
                }
            })
    );

    return field;
}
```

　しかしこの方法はスマートでないと感じます。
　どうせ ``ObservableField`` も同じような概念のオブジェクトで、View が購読開始-終了をしているにすぎないはずなので、同じタイミングで、``rx.Observable<T>`` の subscribe/unsubscribe をさせてあげれば良いはずです。

　ということで作ってみたのがこの ``rx.Observable<T>`` を ``ObservableField<T>`` に変換するクラス。

```java
import android.databinding.ObservableField;

import java.util.HashMap;
import java.util.Map;

import rx.Observable;
import rx.Subscription;
import rx.functions.Action1;

public class RxField<T> extends ObservableField<T> {

    private final Observable<T> observable;
    private final Map<Integer, Subscription> sucscriptionMap = new HashMap<Integer, Subscription>();

    public RxField(Observable<T> observable) {
        super();
        this.observable = observable;
    }

    public RxField(Observable<T> observable, T defaultValue) {
        super(defaultValue);
        this.observable = observable;
    }

    @Override
    public synchronized void addOnPropertyChangedCallback(OnPropertyChangedCallback callback) {
        super.addOnPropertyChangedCallback(callback);

        sucscriptionMap.put(callback.hashCode(), observable.subscribe(new Action1<T>() {
            @Override
            public void call(T value) {
                set(value);
            }
        }));
    }

    @Override
    public synchronized void removeOnPropertyChangedCallback(OnPropertyChangedCallback callback) {
        if (sucscriptionMap.containsKey(callback.hashCode())) {
            final Subscription subscription = sucscriptionMap.get(callback.hashCode());
            subscription.unsubscribe();
            sucscriptionMap.remove(callback.hashCode());
        }

        super.removeOnPropertyChangedCallback(callback);
    }

    @Override
    public void set(T value) {
        // TODO should be readonly, because cannot set value to observable
        super.set(value);
    }

    public Observable<T> tObservable() {
        return observable;
    }
}
```


　``ObservableField`` は、View から購読されると ``addOnPropertyChangedCallback`` が呼ばれ、購読解除されると ``removeOnPropertyChangedCallback`` が呼ばれます(るはずです)。

　なので、このタイミングで ``rx.Observable<T>`` を ``subscribe()``、``subscription.unsubscribe()`` してあげます。購読者(View)が複数になる可能性があるので、 subscription は Map で管理しています。

　で、``rx.Observable<T>`` の値が変わった時(``onNext()``)に、``ObservableField`` の ``set(value)`` を呼んであげれば、``ObservableField`` 側の変更通知(``notifyChanged``)が飛んで、View が更新されます。

　使い方はこんな感じで → [StopWatchSample/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/viewmodel/MainViewModel.java](https://github.com/amay077/StopWatchSample/tree/qiita_20160226/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/viewmodel/MainViewModel.java#L51-L67)
　
## 双方向には対応してません

　この実装は、``rx.Observable`` の更新を ``ObservableField`` 通知するだけです。逆方向（``ObservableField`` の変更を ``rx.Observable`` に適用する）は対応していません。そもそも ``rx.Observable`` は値を設定できないので、それをしたければ ``rx.Observable`` の代わりに ``rx.Subject`` が必要です。

[Android Data Binding + MVVMパターンのサンプルを書いてみた](http://qiita.com/amay077/items/b5c788bb3ee9ff84d9b4) で作成したアプリに、これを適用してみたので、ご参考まで。

* [StopWatchSample/StopWatchAppAndroid - github](https://github.com/amay077/StopWatchSample/tree/qiita_20160226/StopWatchAppAndroid)
