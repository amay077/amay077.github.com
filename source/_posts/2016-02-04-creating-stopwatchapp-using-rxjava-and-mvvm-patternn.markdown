---
layout: post
title: "RxJava + MVVM パターンで作るストップウォッチアプリ"
date: 2015-12-24 23:59:59 +0900
comments: true
categories: [RxJava, Android, ReactiveX, MVVM, Xamarin]
---
　これは [RxJava Advent Calendar 2015 24日目](http://qiita.com/advent-calendar/2015/rxjava) の記事です。

先日、

* [JXUGC #9 Xamarin.Forms Mvvm 実装方法 Teachathon を開催しました - Xamarin 日本語情報](http://ytabuchi.hatenablog.com/entry/2015/12/20/012007)

というイベントがありまして、エクセルソフトの田淵さんが作成したストップウォッチのアプリケーション(注:田淵さんはプログラマではないｗ)を、MVVM識者の方々が「MVVMとしてはこうあるべきだ」と叩きまくる、という恐ろしい?ものでした。
<!--more-->

私はこの勉強会には参加できなかったのですが、ストリーミングとか見て、

<blockquote class="twitter-tweet" lang="ja"><p lang="ja" dir="ltr">僕もストップウォッチ作ってみるかー</p>&mdash; ジェットあめいカスタム (@amay077) <a href="https://twitter.com/amay077/status/677561989359472640">2015, 12月 17</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

などとつぶやいたらご指名されてしまいました(^^)
このイベントは Xamarin を使ったアプリ製作でしたが、せっかくなので **RxJava + MVVM** で作ってみました。
(ご指名に応えないといけないのでその後 Xamarin版も製作)

# ストップウォッチアプリの仕様

上記リンクからの引用です。

* Start/Stop ボタン、Lap ボタン
* StartするとラップボタンはEnable.ストップするとDisable
	* スタートしてからの経過時間をXX'XX.XXXみたいな感じで表示
* 履歴をListViewで残す
* ストップしたら結果をダイアログで出して分岐？
* 今までのラップよりMin, Maxなどをダイアログに表示して次のページに遷移 ←ここ勝手に Toast に仕様変更しましたｗ
* スイッチの切り替えで、ミリ秒の桁を表示/非表示

こんな機能を満たすサンプルを

1. RxJava を使った Android アプリ(Java言語)
1. Reactive Extensions, ReactiveProperty を使った Android アプリ(Xamarin, C#言語)
1. Reactive Extensions, ReactiveProperty を使った iOS アプリ(Xamarin, C#言語)

でそれぞれ作ってみました。

# とりあえず、作ったもの

![つくったもの](https://dl.dropboxusercontent.com/u/264530/qiita/rxjava_mvvm_stopwatch_00.gif)

左は Android-Java製、右は Xamarin.iOS製です。(Xamarin.Android製は省略)

# 1. RxJava を使った Android アプリ(Java言語)

## Model-ViewModel-View(MVVM) で考える

構成図っぽいものを描くとこんな感じになります。

![クラス図的なの](https://dl.dropboxusercontent.com/u/264530/qiita/rxjava_mvvm_stopwatch_01.png)

### Model

　この仕様だと、ストップウォッチの一通りの機能を満たすクラスが Model になります。これを ``StopWatchModel`` という名前にしました。
  
  **「ロジック」は、すべてこの層（このクラス）に書きます。**
　
　例えば、ストップウォッチのタイマーを実行するには、 RxJava で ``Observable.interval`` としますが、これを ViewModel層に書いたら「負け」です。
　
また、ストップウォッチの実行は、画面の表示/破棄と連動しなくてよい(画面遷移しても計測し続けるべき)なので、StopWatchModel の生存期間は、アプリケーションの起動時から終了まで、という事になります。

　RxJava を全面的に使いたいので、 StopWatchModel のプロパティは全て ``Observable<T>`` にしました。RxJava を使わなかったらプロパティではなくコールバックですね。
　何かメソッドを実行したら、その結果は全て ``Observable<T>`` を通じて通知される仕組みです。なので原則として Model のメソッドの戻り値は ``void`` です。

### ViewModel

　しつこいようですが **ここにロジックを書いたら負け** です。
個人としては、条件分岐もしたくない、変数宣言もしたくない、くらいのつもりでいます。もし書いてしまったら「それはModelの方が適切ではないか？」を検討します。

　ViewModel の役割は、Model のプロパティ(コールバック)を、View用に変換して流すこと、Viewのための機能をコマンドとして公開することです。

　例えば、仕様の内、

> スイッチの切り替えで、ミリ秒の桁を表示/非表示

　が、「View用に変換」の良い例になります。
　私の実装では、ミリ秒の桁を表示するか否かの bool 値を、format関数の書式文字列に変換しています。(View側で format して表示しています。)
　↓のような感じです。

```java
/** 時間の表示フォーマット */
public final Observable<String> timeFormat; // field


this.timeFormat = _stopWatch.isVisibleMillis.map(visible -> 
    visible ? "mm:ss.SSS" : "mm:ss");
```

RxJava で「変換」とくれば、 ``map`` など、 ``Observable`` の投影系のメソッドの出番となります。

ViewModel が公開するプロパティも、基本的には ``Observable<T>`` になりました。(これはこのアプリの仕様上、OneWayバインディング＜=Modelによるデータの変化をViewに表示する＞だけで済んだためです。TwoWayバインディング＜=Viewからのデータの入力を受け付ける＞が必要な場合は、``Subject``など、データをセットできる機能が必要になります。)

コマンドとは、Modelのメソッドを呼ぶためのものですが、それに加えて「そのコマンドが実行可能か？」を示すフラグも持ちます。さらにこのフラグも ``Observable<boolean>`` で表します。
こうする事で、「機能が利用可能な時のみボタンを Enable にする」のようなバインディングが可能になります。今回の仕様で言えば

> StartするとラップボタンはEnable.ストップするとDisable

に該当します。

コマンドのインターフェースは↓のようになります。

```java
public interface Command {
    /** このコマンドが実行可能かを示すフラグの更新を通知するObservable */
    Observable<Boolean> canExecuteObservable();

    /** このコマンドの処理を実装する */
    void execute();
}
```

今回は、このインターフェースを ViewModel で匿名クラスを作ることで実装しました。↓のような感じです。この ``commandLap`` をラップボタンとバインドさせます。

```java
/** 経過時間の記録 */
public final Command commandLap = new Command() {
    @Override
    public Observable<Boolean> canExecuteObservable() {
        return _stopWatch.isRunning; // 実行中のみ記録可能
    }

    @Override
    public void execute() {
        _stopWatch.lap();
    }
};
```

あ、ViewModel は View とは疎結合に作ります。Viewを参照してはいけないのはもちろん、``TextView`` や ``Activity`` などが import されていたら「負け」です。

他には、Viewの状態を保持する役割も担いますが、本アプリの仕様では、それに該当する処理はありませんでした。

### View

　View層で行うことは、画面要素のレイアウトとViewModelとのバインディングです。それ以外の事は行いません。.NETの世界では、View層において値の変換を行う機能=ValueConverterが存在しますが、ValueConverterを使うべきかViewModelで行うべきかでよく議論になります。
　
　バインディングの実体は、ViewModelのプロパティである ``Observable<T>`` を ``subscribe`` して、Viewのプロパティにセットしているだけです。前述の通り今回は TwoWay は無いので楽です。TwoWay が出てくるとバインディングのフレームワークにお願いした方がよいです。
　
　例えば、 ``Observalbe<String>`` と TextViewのtextプロパティのバインディングは、下のようになります。

```java
public TextViewBinder toTextOneWay(Observable<String> prop) {
    _subscriptions.add(
        prop.observeOn(AndroidSchedulers.mainThread())
        .subscribe(x -> _textView.setText(x)));

    return this; // メソッドチェーンで連続して呼べるようにしてるだけ
}
```

.NETの世界では、このバインディングを画面定義ファイル(.xaml)に直接記述できます。
Androidでも一部のライブラリや、[今後公式にデータバインディングがサポートされる模様](http://developer.android.com/intl/ja/tools/data-binding/guide.html)ですが、xml でのバンディングの記述は、デバッグしづらくなるので個人的にはそれほどメリットを感じないです。デザイナーとの分業と言っても別な理由で不可能なケースが多いと思います。

## 画面遷移や Toast の表示は誰の責務？

大抵の MVVMフレームワーク に備わっている ``Messenger`` という機能を使います。Android界隈の人には「EventBus」と言った方がわかりやすいかも知れません。

ViewModelが「画面遷移を要求するメッセージ」を投げ、それをViewが受信して画面遷移を行います。

```java メッセージ送信側(MainViewModel.java)
public final Command commandNextView = new Command() {
    @Override
    public void execute() {
        // LapActivity へ遷移させる
        // ほんとは LapViewModel.class を指定すべき(LapActivity は使いたくない)
        messenger.send(new StartActivityMessage(LapActivity.class));
    }
};
```

```java メッセージ受信側(MainActivity.java)
// 画面遷移のメッセージ受信
_viewModel.messenger.register(StartActivityMessage.class.getName(), new Action1<Message>() {
    @Override
    public void call(final Message message) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                final StartActivityMessage m = (StartActivityMessage)message;
                Intent intent = new Intent(MainActivity.this, m.activityClass);
                MainActivity.this.startActivity(intent);
            }
        });
    }
});
```

今回は簡単な Messenger を実装しました。VM->Vの通知にしか使わないのでVM毎に一つ持つようにしています。

# 2.3. Reactive Extensions, ReactiveProperty を使った Android/iOS アプリ(Xamarin, C#言語)

[Xamarin](https://xamarin.com/) は、 C# で Android/iOS が作れるプロダクトです。
RubyMotion のように、CocoaTouch や Android SDK の API をラップし、同じ名称のクラス,メソッドで C# から呼び出せるようにしています。

* [Xamarin 日本語情報](http://ytabuchi.hatenablog.com/)
* [マカーの人が Xamarin について勘違いしていそうな５つのこと - Qiita](http://qiita.com/amay077/items/2e86b44e5f274a34b2e9)

.NETのオープンソース実装である mono 由来の製品であり、また Microsoft とのパートナーシップも結んでいることから、.NET の資産の多くが利用可能です。

何が言いたいかと言うと、RxJava も MVVM パターンも、元は .NET のアプリケーション開発の分野で発案・成熟してきた考え方であり、豊富な.NET製ライブラリ(今回だと Reactive Extensions と ReactiveProperty)を使って Android/iOS アプリを開発できる、という事です。

## Model-ViewModel-View(MVVM) で考える

Xamarin でも MVVM の役割はまったく同じですが、
**「Model-ViewModel を Android/iOS で使いまわせる」** 
という大きなメリットがあります。

Model と ViewModel からは、プラットフォームに依存するコードは排除できます(すべきです)。 
Xamarin(というか .NET) ではプラットフォーム非依存の処理をライブラリ化できます(これを PCL=Portable Class Library と言います)。

Android と iOS でそれぞれに実装が必要なのは、View と、そのバインディングのみです。

![クラス図的なの](https://dl.dropboxusercontent.com/u/264530/qiita/rxjava_mvvm_stopwatch_02.png)

## Reactive Extensions について

本家[Rx.NET](https://github.com/Reactive-Extensions/Rx.NET)です。RxJava はこの Reactive Extensions を Java にポートしたものです。
RxJava には、いくつか便利なメソッドが追加されています(``compose`` とか)が、殆ど同じです。
また、 C# はラムダ式を標準でサポートしていることから、 retrolambda などに頼らなくても見やすいコードが書けるのは言うまでもないでしょう。

## ReactiveProperty について

[ReactiveProperty](https://github.com/runceel/ReactiveProperty/blob/master/README-ja.md) は、Rxの機能を活かしてMVVMパターンの実装を手助けしてくれるライブラリです。

* [MVVMとリアクティブプログラミングを支援するライブラリ「ReactiveProperty v2.0」オーバービュー - かずきのBlog@hatena](http://blog.okazuki.jp/entry/2015/02/22/212827)

Java版ストップウォッチでは、``StopWatchModel`` や ViewModel のプロパティを全て ``Observable<T>`` としましたが、 Xamarin版では ``ReactiveProperty<T>`` としています。 ``ReactiveProperty<T>`` は ``Observable<T>`` から継承しているので、それほど大差はありませんが、``Subject`` のように値の設定をサポートしていたり、バリデーション、エラー通知の仕組みが備わっています。

また、ReactiveProperty は、 Android の View要素とのバインディング機能も持ちます。これを使うとバインディングが以下のように書けます。

```csharp
// TextView(textTime) と viewModel.Time のバインド
FindViewById<TextView>(Resource.Id.textTime)
    .SetBinding(v => v.Text, 
        _viewModel.Time.Select(x => x.ToString())
        .ObserveOnUIDispatcher()
        .ToReactiveProperty());
```

iOS のバインディングはありませんが、 ~~ソースの一部を持ってくる事で、殆ど解決します~~ [ツイートしたら取り込んでもらえました(^^)](https://twitter.com/okazuki/status/679256704689684480) 。

# 作ったアプリのソース

* [amay077/StopWatchSample](https://github.com/amay077/StopWatchSample)

それぞれ、

1. [RxJava を使った Android アプリ(Java言語)](https://github.com/amay077/StopWatchSample/tree/master/StopWatchAppAndroid)
1. [Reactive Extensions, ReactiveProperty を使った Android アプリ(Xamarin, C#言語)](https://github.com/amay077/StopWatchSample/tree/master/StopWatchAppXamarin/StopWatchApp.Android)
1. [Reactive Extensions, ReactiveProperty を使った iOS アプリ(Xamarin, C#言語)](https://github.com/amay077/StopWatchSample/tree/master/StopWatchAppXamarin/StopWatchApp.iOS)
1. [Xamarin版アプリの Model, ViewModel](https://github.com/amay077/StopWatchSample/tree/master/StopWatchAppXamarin/StopWatchApp.Core)

にあります。

Java版は、

* [StopWatchModel.java](https://github.com/amay077/StopWatchSample/blob/master/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/models/StopWatchModel.java)
* [MainViewModel.java](https://github.com/amay077/StopWatchSample/blob/master/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/viewmodel/MainViewModel.java)
* [MainActivity.java](https://github.com/amay077/StopWatchSample/blob/master/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/views/MainActivity.java)

を見るとだいたい分かると思います。

また、Java版 と Xamarin版では、

* [StopWatchModel.java](https://github.com/amay077/StopWatchSample/blob/master/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/models/StopWatchModel.java) と [StopWatchModel.cs](https://github.com/amay077/StopWatchSample/blob/master/StopWatchAppXamarin/StopWatchApp.Core/Models/StopWatchModel.cs)
* [MainViewModel.java](https://github.com/amay077/StopWatchSample/blob/master/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/viewmodel/MainViewModel.java) と [MainViewModel.cs](https://github.com/amay077/StopWatchSample/blob/master/StopWatchAppXamarin/StopWatchApp.Core/ViewModels/MainViewModel.cs)
* [MainActivity.java](https://github.com/amay077/StopWatchSample/blob/master/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/views/MainActivity.java) と [MainActivity.cs](https://github.com/amay077/StopWatchSample/blob/master/StopWatchAppXamarin/StopWatchApp.Android/Views/MainActivity.cs) と [MainViewController.cs](https://github.com/amay077/StopWatchSample/blob/master/StopWatchAppXamarin/StopWatchApp.iOS/Views/MainViewController.cs)

あたりを見比べるといいと思います。

# まとめ

MVVM と RxJava はとても相性がよいと感じました。

Model → ViewModel → View と通知を伝搬させるのに、そのまま ``Observable<T>`` を繋げればよいのですから。加工が必要なら ``map`` などのオペレータを挟むだけ。
これがコールバックだったら…恐ろしくて想像したくありません。

Model が使用するDB層やWebAPIなども RxJava をサポートしていたら、もっと便利になると思います。(Realm は RxJava サポートが追加されたようですね！)

View - ViewModel のデータバインディングにも RxJava は有効ですが、こちらは、Android公式の Data Binding がどう実装されるかで未来が変わってきそうです。

Android-Java には、まだメジャーな MVVMフレームワークが無いので、登場が待たれるところです。

# おまけ：反省など

## View か ViewModel か Model か問題

下は View に書かれている「現在時刻と表示書式文字列のどちらかが更新されたら、時刻をフォーマットして流す」という Observable です。

```java
// フォーマットされた時間を表す Observable（time と timeFormat のどちらかが変更されたら更新）
final Observable<String> formattedTime = Observable.combineLatest(
        _viewModel.time,
        _viewModel.timeFormat, (Long time, String format) -> {
            final SimpleDateFormat sdf = new SimpleDateFormat(format, Locale.getDefault());
            return sdf.format(new Date(time));
        });
```

これは、ViewModel に用意すべきだったかも知れません。いやいや、フォーマットされた時間を通知する機能が Model にあってもおかしくないとも言えます。
実際、 ``LapActivity`` でも同じコードを書いているので D.R.Y原則にも反します。やっぱ Model に持たせるべきだったと反省。

## Model に戻り値が void でないメソッドを作っちゃった問題

> 原則として Model のメソッドの戻り値は void です

の原則に反して、戻り値で最速、最遅ラップ値を返してしまいました。
Toast表示のためだけに取得できればいいやと思いこうしたのですが、これでは「最速、最遅ラップを常に画面に表示する」という仕様変更があっただけで破綻します。これは悪手でした、反省。

だいたいラップの最大、最小の取得は、 ``Observable<List<Long>> laps``  を ``map`` で変換すればよいだけの話ですね。LINQ あるいは Stream API が使えれば ``List<Long>`` から min/max を取得するのも簡単ですし。


## Timer を 1ms 間隔にしちゃった問題

``Observable.interval(1ms)`` ってやっちゃいましたが、START の時間を覚えておいて、LAP, STOP された時に、現在時刻との差分を取ればよかったですね。基本的なムダで反省。

[JXUG で話した MVVM の活用の解説を | Moonmile Solutions Blog](http://www.moonmile.net/blog/archives/7627) より
> Lap ボタンを押したタイミングで DateTime.Now を取得すればよいわけで、何も定期的に内部データを更新する必要はありません

その通りですね。。。

## UIスレッドへの変換をだれがやるのか問題

今回は、以下のように、自作したバインディングの中で ``observeOn(AndroidSchedulers.mainThread())`` 行っています。

```java
public TextViewBinder toTextOneWay(Observable<String> prop) {
    _subscriptions.add(
        prop.observeOn(AndroidSchedulers.mainThread())
            .subscribe(x -> _textView.setText(x)));

    return this;
}
```

これを ViewModel で行うこと(ViewModel が公開する Observable は必ずUIスレッドで実行されるというルール)もできます。
が、セオリーが分かっていません。とりあえず View側で observeOn しとけば安全かなと思って上記のようにしているだけです。使用するMVVMフレームワークの仕様にも依存しそうです。

## StopWatchModel のプロパティは Hot？ それとも Cold？

StopWatchModel の各プロパティである ``Observable<T>`` は、 **``subscribe`` をトリガーに値が流れ始めるものではないので Hot** ですね。

また、``BehaviorSubject`` を使っているので、 ``subscribe`` 時には、その時点の最新の値が流れてきます。

シングルトンの ``StopWatchModel`` に対して、 ``MainActivity`` に続いて ``LapActivity`` でも購読した時に、正しくラップタイム群が表示できるのは、``BehaviorSubject`` であるためですね。

``refCount`` してないけど、ちゃんと破棄されているのかは未確認。。。

## Observalbe\<List\<T>>

ラップタイム群を通知するプロパティは ``Observalbe<List<T>>`` にしています。
この場合、List の中身を変更されても通知されないので  ``Collections.unmodifiableList`` で変更不可にしてから onNext で通知しています。LAPボタンが押される度に List を作りなおしている感じになります。

ListView とのバインディングも同じで、onNext を受信する度に、ListView を洗い替えしています。
このムダが嫌、大量データでパフォーマンスに問題が出る場合は、.NET にある [``ObservableCollection<T>``](https://msdn.microsoft.com/ja-jp/library/ms668604(v=vs.110).aspx) のような仕組みを作る必要があります。(Rx.NET, RxJava では管轄外かな？)

``ObservableCollection<T>`` は、リストへの追加、削除、変更をアイテム毎に通知／監視できます(「 *項目X* が *2番目* に *追加* された」のような)。適切な通知とバインディングを実装すれば、ListView の差分更新が可能です（面倒ですが）。


# 参考

* [MVVMのModelにまつわる誤解 - the sea of fertility](http://ugaya40.hateblo.jp/entry/model-mistake)
* [JXUG で話した MVVM の活用の解説を | Moonmile Solutions Blog](http://www.moonmile.net/blog/archives/7627) - タイマを View/ViewModel/Model に持つそれぞれの理由が解説されています。
* [RxJava - Rxで知っておくと便利なSubjectたち - Qiita](http://qiita.com/hide92795/items/f7205c8171826cc2153b)
* [RxJava - Hot Observable と ConnectableObservable について - Qiita](http://qiita.com/amay077/items/4bb6b09a1911b074f50c)
