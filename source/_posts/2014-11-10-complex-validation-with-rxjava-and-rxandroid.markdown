---
layout: post
title: "「チェックAがONならば、項目Bは入力必須とする」という Validation を RxJava + RxAndroid でやる"
date: 2014-11-10 01:32:20 +0900
comments: true
categories: [RxJava, RxAndroid, Android]
---
　例えばショッピングサイトとかの発送先指定のフォーム『登録されている住所とは違う住所に送りたい時、「別の住所に送る」をチェックする、すると「住所2」が必須入力となり、入力するまで次へ進めない』的なちょっと込み入ったValidationをReactive ExtensionsのJava版、[RxJava](https://github.com/ReactiveX/RxJava)と[RxAndroid](https://github.com/ReactiveX/RxAndroid)でやってみました。
<!--more-->
# 動作イメージ

　まずいきなり動作結果から。

![](https://dl.dropboxusercontent.com/u/264530/qiita/complex_validation_with_rxjava_and_rxandroid.gif)

* 住所1は入力必須。
* 住所2は「住所2へ配送する」がチェックされている場合のみ、入力必須。
* 必須項目が入力されていない場合はボタンを押せない

こんな仕様です。

# 実装

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_my);

    // 「注文を確定する」ボタン
    final Button buttonSubmit = (Button)findViewById(R.id.buttonSubmit);

    // チェックボックスのON/OFFをObservable化
    final Observable<Boolean> useSecondaryAddress =
            ViewObservable.input((CheckBox) findViewById(R.id.checkUseSecondary), true)
            .map(new Func1<OnCheckedChangeEvent, Boolean>() {
                @Override
                public Boolean call(OnCheckedChangeEvent onCheckedChangeEvent) {
                    return onCheckedChangeEvent.value;
                }
            });

    // 住所1をObservable化
    final Observable<OnTextChangeEvent> primaryAddress =
            ViewObservable.text((EditText) findViewById(R.id.editPrimaryAddress), true);
    // 住所2をObservable化
    final Observable<OnTextChangeEvent> secondaryAddress =
            ViewObservable.text((EditText) findViewById(R.id.editSecondaryAddress), true);

    // チェックボックスと住所2の必須条件をObservable化
    final Observable<Boolean> secondaryIsValid = 
        Observable.combineLatest(useSecondaryAddress, secondaryAddress,
            new Func2<Boolean, OnTextChangeEvent, Boolean>() {
                @Override
                public Boolean call(Boolean useSecondary, OnTextChangeEvent secondaryAddress) {
                    if (!useSecondary) {
                        return true;
                    }

                    return !TextUtils.isEmpty(secondaryAddress.text);
                }
            });


    // 全部まとめると、
    //  住所1は入力必須、
    //  住所2はチェックボックスがONの時だけ入力必須
    //  必須条件を満たしていたらtrueを流す
    final Observable<Boolean> isValidAll = Observable.combineLatest(primaryAddress, secondaryIsValid,
            new Func2<OnTextChangeEvent, Boolean, Boolean>() {
                @Override
                public Boolean call(OnTextChangeEvent primaryAddress, Boolean isValidSecondary) {
                    if (!isValidSecondary) {
                        return false;
                    }

                    return !TextUtils.isEmpty(primaryAddress.text);
                }
            });


    // 購読、監視
    isValidAll.subscribe(new Observer<Boolean>() {
        @Override
        public void onNext(final Boolean isValid) {
            // 必須条件を満たしていたら「注文を確定する」を有効にする
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    buttonSubmit.setEnabled(isValid);
                }
            });
        }

        @Override
        public void onCompleted() {
        }

        @Override
        public void onError(Throwable e) {
        }
    });
}
```

　``ViewObservable.xxx`` で、UI要素をObservable化します。これはRxAndroidの機能。これでテキストの変更とか、チェックボックスの変更のたびに、``OnNext``が発生するようになります。

　Validationでは、RxJavaの機能である ``Observable.combineLatest``がキモで、こいつに2つのObservableを渡してやると、その片方が値が変化した時に、``T3 call(T1 a, T2 b)`` が呼ばれます。T1、T2 は渡すObservableの型、T3は後続へ流す型で、Validationなので``Boolean``です。
上記 ``secondaryIsValid`` の実装では、「住所2に配送する」のチェックボックスと「住所2」のテキストボックスの2つのObservableを渡していて、

* 「住所2に配送する」がOFFなら ``true`` を返す
* 「住所2に配送する」がONで、且つ「住所2」が空でなければ ``true`` を返す

としています。

　次に、``isValidAll`` の実装では、「住所1」と ``secondaryIsValid`` を渡していて、

* ``secondaryIsValid`` が ``false`` なら ``false`` を返す
* ``secondaryIsValid`` が ``true`` で、且つ「住所1」が空でなければ ``true`` を返す

という実装です。

　んで、こいつ(``isValidAll``)を購読(``subscribe``)すると、``onNext`` にValidationの結果が通知されるので、ボタンの``Enabled``を切り替えます。

# まとめ

　このレベルだと、すべてのUI要素に変更通知を仕込んで共通な関数を呼ぶ、的な実装で問題ないですが、要素や条件が増えてくると大変です。

　Observable と combineLatest を使うと、制約の一部を(Observableに)部分化できて、それらを組み合わせるのも自由自在(Observableだから)。

Javaなのでかなり長ったらしくて読みづらいコードになってしまいました。

**[Xamarin.Android](http://xamarin.com/) + [本家Reactive Extensions](https://rx.codeplex.com/) + [ReactiveProporty](https://reactiveproperty.codeplex.com/) なら、相当スッキリするんだけどなあー**