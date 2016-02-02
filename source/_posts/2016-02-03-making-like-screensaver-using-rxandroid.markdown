---
layout: post
title: "RxAndroid でスクリーンセーバー的な機能を作る"
date: 2014/12/09 00:00:00 +0900
comments: true
categories: [RxJava, RxAndroid, Android]
---
　これは [Android Advent Calendar 2014 8日目](http://qiita.com/advent-calendar/2014/android) の記事です。

　例えば◯秒間操作がなかったらパスキーロック画面を表示する、とかそういうの。普通に作るとタイマーを使って面倒な感じになっちゃいますが、[RxJava](https://github.com/ReactiveX/RxJava) と [RxAndroid](https://github.com/ReactiveX/RxAndroid) を使うととても簡単にできます。
<!--more-->

## RxJava + RxAnroid の場合

　例えば、画面に ``EditBox`` と ``Button`` があって、「文字列の入力」と「ボタンが押されたか」を監視、◯秒間操作がなかったら××する、という処理をしたい時、RxJava+RxAndroid では以下のように書けます。

```java MyActivity.java
public class MyActivity extends Activity {
    private static final String TAG = "MyActivity";
    private Subscription _subscription;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my);

        final EditText editName = (EditText)findViewById(R.id.editName);
        final View buttonOk = findViewById(R.id.buttonOk);

        // OnTextChangeEvent や OnClickEvent をただの Void シグナルに変換
        final Func1<Object, Void> signalizer = new Func1<Object, Void>() {
            @Override
            public Void call(Object onClickEvent) {
                return null;
            }
        };

        // 文字入力イベントのストリームと…
        _subscription = ViewObservable.text(editName).map(signalizer)
                // ボタン押されたのストリームを合体
                .mergeWith(ViewObservable.clicks(buttonOk).map(signalizer))
                // 3秒間なんもなかったらエラーにする
                .timeout(3, TimeUnit.SECONDS)
                .subscribe(new Action1<Void>() {
                    @Override
                    public void call(Void dummy) {
                        // 何かアクションがあったらこっち
                        Log.d(TAG, "文字が入力されたか、ボタンが押されたよ");
                    }
                }, new Action1<Throwable>() {
                    @Override
                    public void call(Throwable throwable) {
                        // 3秒間何もなかったらこっち
                        runOnUiThread(new Runnable() {
                            @Override
                            public void run() {
                                Toast.makeText(MyActivity.this, 
                                    "３秒間何も操作がありませんでした", Toast.LENGTH_SHORT)
                                    .show();
                            }
                        });
                    }
                });
    }

    @Override
    protected void onDestroy() {
        // イベント系は無限ストリームだから開放してやらないとリークするはず
        _subscription.unsubscribe();
        super.onDestroy();
    }
}
```

　``ViewObservable.text(editName)`` がテキストが入力される度にシグナルを発するストリーム、``ViewObservable.clicks(buttonOk)``がボタンが押される度にシグナルを発するストリームです。これらを [``mergeWith``](http://rxmarbles.com/#merge) で合体させます。

　あとは [``timeout``](https://github.com/ReactiveX/RxJava/wiki/Filtering-Observables#timeout) につなげるだけ。３秒以内にシグナルがあったら onNext→``new Action<Void>()``のとこ、3秒以上何も操作がなかったらタイムアウトして onError→``new Action<Throwable>()`` のとこに飛びます。あとはご自由に、ここでは ``Toast`` を表示してるだけです。

　注意点は、イベントから生成されたストリームは無限、つまり ``onComplete`` は来ない。こういう ``Observable`` は自力での購読解除（``unsubscribe``）が必須です。

これを動かすとこんな感じになります

![](https://dl.dropboxusercontent.com/u/264530/qiita/make_screensaver_using_rxjava_01.gif)
## Xamarin.Android + Rx本家の場合

　さて Xamarin です。Xamarin では本家の [Reactive Extensions](https://rx.codeplex.com/) が使用できます。RxAndroid と同じことをやると下のように書けます、スマート。

```csharp MainActivity.cs
[Activity(Label = "RxJavaSample", MainLauncher = true, Icon = "@drawable/icon")]
public class MainActivity : Activity
{
    protected override void OnCreate(Bundle bundle)
    {
        base.OnCreate(bundle);

        // Set our view from the "main" layout resource
        SetContentView(Resource.Layout.Main);

        var editName = FindViewById<EditText>(Resource.Id.editName);
        var buttonOk = FindViewById<Button>(Resource.Id.buttonOk);

        Observable.FromEventPattern<TextChangedEventArgs>(editName, "TextChanged").Select(_=>true)
            .Merge(Observable.FromEventPattern(buttonOk, "Click").Select(_=>true))
            .Timeout(TimeSpan.FromSeconds(3))
            .Subscribe(_ => {} , 
            e => RunOnUiThread(() => Toast.MakeText(this, 
                "３秒間何も操作がありませんでした", ToastLength.Short).Show()));
    }
}
```

## まとめ

　Reactive Extensions を使うと、UIイベントをストリームに変換でき、合成・加工・フィルタなどして様々な応用ができます。しかしこれは Rx のパワーのまだ半分。もう半分は、WebAPI とか DB とか、Model 由来のレスポンスもストリーム化できること。どちらも Observable にしたら、あとはそれをつなぐだけでアプリ完成！
　さあみんなで Rx にロックインされましょう！
