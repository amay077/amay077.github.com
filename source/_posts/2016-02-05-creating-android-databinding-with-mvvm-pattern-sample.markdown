---
layout: post
title: "Android Data Binding + MVVMパターンのサンプルを書いてみた"
date: 2016-01-28 23:59:59 +0900
comments: true
categories: [Android, MVVM, DataBinding, RxJava]
---

``notifyPropertyChanged`` とか、[どこかで見たことのある](https://ufcpp.wordpress.com/2009/12/28/inotifypropertychanged-%E3%81%AE%E5%AE%9F%E8%A3%85/)機能が満載の [Android Data Binding](http://developer.android.com/intl/ja/tools/data-binding/guide.html) ですが、登場以来あまり追えてなかったのでやっとサンプルをつくってみました。
<!--more-->

といっても

* [RxJava + MVVM パターンで作るストップウォッチアプリ - Qiita](http://qiita.com/amay077/items/8464a22e3063642112ed)

で作ったストップウォッチアプリを Data Binding 化しただけです。

[前回](http://qiita.com/amay077/items/8464a22e3063642112ed#model-viewmodel-viewmvvm-%E3%81%A7%E8%80%83%E3%81%88%E3%82%8B) との違いを図に示します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/rxjava_mvvm_stopwatch_03.png)

* View-ViewModel で全面的に使用していた ``rx.Observable<T>`` の代わりに、``ObservableField<T>`` を使用。
* View側で「オレオレDataBinding」を実装していた箇所を、Android の Data Binding に置き換え。つまりバインディングの定義はレイアウトxmlへ記述。
* Model は相変わらず ``rx.Observable<T>`` のまま。なので ViewModel で ``rx.Observable<T>`` → ``ObservableField<T>`` へ変換。
* メソッドとのバインドに ``Command`` を使用していたが、Android Data Binding の [Binding Events](http://developer.android.com/intl/ja/tools/data-binding/guide.html#binding_events) に置き換え。
* ListView とデータ群のバインディングの方法が分からなかったので、[カスタムBinding](http://developer.android.com/intl/ja/tools/data-binding/guide.html#custom_setters)で対応。(listItem のバインディングじゃなくて、リストの件数の増減を反映させるやつ。)
* ArrayAdapter 使ってたんだけどこいつは Binding に対応していない？ので Adapter を自作。 

## MainActivity のバインディングの定義

``activity_main.xml`` はこんな感じ。

``@{ }`` で  ``MainViewModel`` に用意した ``ObservableField<T>`` または、イベントハンドラとバインドしてます。

```xml activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable name="viewModel"
            type="com.amay077.stopwatchapp.viewmodel.MainViewModel"/>
    </data>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
        android:orientation="vertical"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingTop="@dimen/activity_vertical_margin"
        android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">

        <TextView android:id="@+id/textTime"
            tools:text="00:00.000"
            android:text="@{viewModel.formattedTime}"
            android:textSize="50sp"
            android:gravity="center_horizontal"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

        <Button
            android:id="@+id/buttonStartStop"
            android:text="@{viewModel.runButtonTitle}"
            android:onClick="@{viewModel.onClickStartOrStop}"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
        <Button
            android:id="@+id/buttonLap"
            android:text="Lap"
            android:enabled="@{viewModel.isRunning}"
            android:onClick="@{viewModel.onClickLap}"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
        <Switch
            android:id="@+id/switchVisibleMillis"
            android:checked="@{viewModel.isVisibleMillis}"
            android:onClick="@{viewModel.onClickToggleVisibleMillis}"
            android:text="小数点以下を表示"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

        <ListView
            android:id="@+id/listLaps"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:formattedLaps="@{viewModel}" />
    </LinearLayout>
</layout>
```

ListView で ``app:formattedLaps="@{viewModel}"`` としているところだけが特殊で、これは ``MainActivity.java`` に定義したカスタムSetter を呼び出します。

``MainActivity.java`` はこんな感じ。

```java MainActivity.java
public class MainActivity extends AppCompatActivity {

    private /* final */  MainViewModel _viewModel;
    private CompositeSubscription _subscriptions = new CompositeSubscription();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        final ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);

        _viewModel = new MainViewModel(this.getApplicationContext());
        binding.setViewModel(_viewModel);

        // ■ViewModel からの Message の受信（省略）
    }

    /**
     * ListView と ViewModel のカスタムバインディング
     *
     * TODO 本当は viewModel.formattedLaps とバインドしたい
     */
    @BindingAdapter("formattedLaps")
    public static void setFormattedLaps(ListView listView, final MainViewModel viewModel) {
        final LapAdapter adapter = new LapAdapter(listView.getContext());
        listView.setAdapter(adapter);
        
        // formattedLaps が変化した時に呼ばれるイベントで、Adapterを洗い替え。
        viewModel.formattedLaps.addOnPropertyChangedCallback(new android.databinding.Observable.OnPropertyChangedCallback() {
            @Override
            public void onPropertyChanged(android.databinding.Observable sender, int propertyId) {
                adapter.clear();
                adapter.addAll(viewModel.formattedLaps.get());
            }
        });

        // バインド時に値を更新
        adapter.clear();
        adapter.addAll(viewModel.formattedLaps.get());
    }

    @Override
    protected void onDestroy() {
        _viewModel.unsubscribe();
        super.onDestroy();
    }
}
```

オレオレBindingがごっそり消えてスッキリ。
``setFormattedLaps`` がカスタムSetterで、この中で ``MainViewModel.formatterLaps`` を監視し、値が変わったら Adapter を総入れ替えしてます。が、これが正しいやり方かわからない。
[extensions/baseAdapters/src/main/java/android/databinding/adapters](https://android.googlesource.com/platform/frameworks/data-binding/+/android-6.0.0_r7/extensions/baseAdapters/src/main/java/android/databinding/adapters) にはそれらしいのがないでござるよ。。。

## ViewModel 側

この辺みてください。大したことはやってないです。（急に雑になったw）

* [StopWatchSample/MainViewModel.java](https://github.com/amay077/StopWatchSample/blob/android_data_binding_v1_20160128/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/viewmodel/MainViewModel.java)
* [StopWatchSample/ObservableUtil.java](https://github.com/amay077/StopWatchSample/blob/android_data_binding_v1_20160128/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/viewmodel/ObservableUtil.java)

``ObservableUtil.toObservableField`` とか、もうどっかの誰かがやってそうだし、事実上標準の何かが出てきそうな気がすごくします。

## おまけ

### Messenger を RxJava ベースにした

* [OttoからRxJavaへの移行ガイド - Qiita](http://qiita.com/yyaammaa/items/57d8baa1e80346e67e47)
* [Android - RxJavaでEventBusを作った - Qiita](http://qiita.com/kubode/items/a4ece37834446c9a39c8)

らしいので、自作してた ``Messenger`` を [RxJava ベースにしてみました](https://github.com/amay077/StopWatchSample/blob/android_data_binding_v1_20160128/StopWatchAppAndroid/app/src/main/java/com/amay077/stopwatchapp/frameworks/messengers/Messenger.java)。
ViewModel→Viewの通知
にしか使ってないので、あまり ``rx.Observable<T>`` にする旨味はなかったですね。あ、``ofType`` って便利ですね。

## まとめ

今回作ったアプリの全ソースは

* [StopWatchApp.Android](https://github.com/amay077/StopWatchSample/tree/android_data_binding_v1_20160128/StopWatchAppXamarin/StopWatchApp.Android)

です。

.NETアプリケーション開発では、ViewModel を View にバインドすることが殆どなので、典型的な例としてやってみました。

レイアウトに直接バインドを定義できるので、コードビハインド(Javaのソース)はスッキリしますが、個人的にはあまり好きではありません。
コードビハインドに(``textTime.SetBinding(v => v.Text, viewModel.Time)`` みたく)書いた方が、定義情報がまとまっていて管理しやすい、デバッグしやすいと思うからです。（同じ理由で、xmlに直接記述する [Expression Language](http://developer.android.com/intl/ja/tools/data-binding/guide.html#expression_language) も好きではありません。）
が、今のところ、Android Data Binding では、レイアウトXMLでしかバインディングを定義できないようですね。

ともあれ、[AndroidBinding](https://github.com/gueei/AndroidBinding) とか Butter Knife はこれで駆逐されていく（前者はすでに息してなさそうですが）と思うので、新しいアプリ開発では積極的に使っていこうかなと思います。

## 参考

* [Data Binding Guide | Android Developers](http://developer.android.com/intl/ja/tools/data-binding/guide.html)
* [Android - Butter Knife、今までありがとう。 Data Binding、これからよろしく。 - Qiita](http://qiita.com/izumin5210/items/2784576d86ce6b9b51e6#after-listview)
* [[Android] – Data Bindingつかってみた – NET BIZ DIV. TECH BLOG](http://tech.recruit-mp.co.jp/mobile/android-data-binding/)
