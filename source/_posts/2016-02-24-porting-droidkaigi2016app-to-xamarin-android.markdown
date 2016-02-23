---
layout: post
title: "DroidKaigi2016アプリを Xamarin.Android に移植した話"
date: 2016-02-24 00:19:01 +0900
comments: true
categories: [Android, Java, Xamarin, DroidKaigi]
---
　DroidKaigi2016 の開催前、[公式アプリが有志によって開発中](https://github.com/konifar/droidkaigi2016) とのツイート(だったかな？)を見て、ふとこれを「Xamarin.Android に移植してみよう」と思い、夜な夜なぼちぼちと始めました。
<!--more-->

　後付けですが、移植するにあたり調査したかったのは主に、

* Android-Java の OSSライブラリがどのくらい Xamarin.Android でも利用可能か？
* (勉強をサボっていた)Xamarin.Android での Material Design の適用方法

です。

## Xamarin.Android について(知らない人向け)

　Xamarin.Android は、Android API(Javaクラスライブラリを含む)の薄いラッパーで、クラス・メソッド名などは殆どそのままに、言語が Java から C# になったようなものです。
　なので、 ``activity_main.xml`` などのリソースファイルもほぼそのまま転用可能です。

* [Xamarin.Android で作った HelloWorld のソースを眺めてみる](http://qiita.com/amay077/items/3232064cc8880c809aee)

　尚、 Xamarin.Forms というワンソースで複数プラットフォームで動作するアプリを開発できるフレームワークとは別のものです。

## Android プロジェクトの Xamarin.Android への移植方法

すごく大雑把に、以下のような手順で移植します。

1. Androidプロジェクト(以下 Java と表記)の ``/res`` 以下を Xamarin.Androidプロジェクト(以下 Xamarin)配下にコピー
2. Java のソースコード群を、 package構成を崩さずに Xamarin.Android で再構成（結局のところコードの書き直し）
3. Java側で使われているOSSライブラリと同等のものを、nuget・Xamarin Components で探してXamarin側に追加（なければ ``.jar`` ファイルを入手して Xamarin で使えるように Binding Library を作成）
4. あとはひたすら try and error and error and error...

## Android Data Binding を、Xamarin ではどうしたか？

　DroidKaigi2016 のアプリには [DataBinding](http://developer.android.com/intl/ja/tools/data-binding/guide.html) が使われています。ただ、 ``BaseObservable`` や ``ObservalbeField`` によるガッツリとした OneWay/TwoWay のデータバインディングではなく、POJOなデータクラスを使う [OneTime](https://msdn.microsoft.com/ja-jp/library/system.windows.data.bindingmode(v=vs.110).aspx) なものしかなかったので、Xamarin への移植に際しては [ReactiveProperty](https://github.com/runceel/ReactiveProperty/blob/master/README-ja.md) や、 [MVVMCross](https://github.com/MvvmCross/MvvmCross) などのデータバインディング機能に頼る必要はありませんでした。

　一方、Android Data Binding のもう一つの(副次的な)機能である View binding(``findViewById`` が要らなくなるアレ)の対応は大変でした。

　まず、``activity_main.xml`` などのデータバインド範囲を括る ``<layout></layout>`` ですが、このタグは Xamarin Studio は解釈してくれないのでエラーになります。このタグはもれなくコメントアウトが必要でした。また、カスタムデータバインディングが使われている箇所も同じくです。
　なので当然、Android Studio(gradle)が生成する ``DataBinding`` クラスも使用できません。
　仕方ない(というか始めからわかっていましたが) ``ActivityMainBinding`` などに相当するクラスを必要を満たす範囲で自作しました。レガシーな ``FindViewByID()`` を使って。

　Windowsアプリ開発の世界では、「DSL で記述された画面レイアウトからUI要素変数を自動生成する」ことは、IDE である Visual Sutdio が普通に行ってくれます。Xamarinアプリ開発のIDEである Xamarin Studio も、iOS の ``.storyboard`` ファイルを読んで、自動的に ``HogeViewController.designer.cs`` にUI要素変数を生成してくれます。
　Xamarin.Android でも ``MainActivity.designer.cs`` とか生成してくれてもいいのになー、とは頭の片隅で思い続けています。(自作Plug-inとかでなんとかできるのかな？)

## DroidKaigi2016 で使われているJavaライブラリを、Xamarinではどうしたか？

　DroidKaigi2016アプリでは非常にたくさんのOSSライブラリが[使用されており](https://github.com/konifar/droidkaigi2016#libraries)、それを眺めるだけでも非常に勉強になります。このソースを読んで初めて知ったものが何個もありました。
　アプリを Xamarin.Android へ移植するにあたり、これらにどのように対応したかを記します。

### Android Support Libraries

 これらは、nuget パッケージが用意されています。要注意なのは、Xamarin Component にも[同じものが存在](https://components.xamarin.com/view/xamandroidsupportdesign)していて、大抵はそちらの方が古くて動かない、ということです。
 
* [Xamarin.Android.Support.CustomTabs](https://www.nuget.org/packages/Xamarin.Android.Support.CustomTabs/)
* [Xamarin.Android.Support.Design](https://www.nuget.org/packages/Xamarin.Android.Support.Design/)
* [Xamarin.Android.Support.v4](https://www.nuget.org/packages/Xamarin.Android.Support.v4/)
* [Xamarin.Android.Support.v7.AppCompat](https://www.nuget.org/packages/Xamarin.Android.Support.v7.AppCompat/)
* [Xamarin.Android.Support.v7.CardView](https://www.nuget.org/packages/Xamarin.Android.Support.v7.CardView/)
* [Xamarin.Android.Support.v7.RecyclerView](https://www.nuget.org/packages/Xamarin.Android.Support.v7.RecyclerView/)

### [Dagger2](http://google.github.io/dagger/)

　Dependency Injection を Annotation ベースで行うライブラリ。
　これはないかなーと思いましたがありました。Dagger(短剣) に対して、その名も Stiletto(短剣)ｗ

* [Stiletto](http://stiletto.bendb.com/) - Stiletto is a .NET port of Dagger, the lightweight Android dependency injector from Square.

　使い方も殆ど一緒。どうも Dagger1 相当の機能のようですが、アプリ側は少しの修正で対応できました。
　もっとも Dagger すら使ったことがなかったので、その理解に少々時間を要しました。
　Stiletto は、Xamarin.iOS でも使えるようですが、残念ながら PCL対応していなさそう。PCL対応のプルリクを送るのは今後やってみたいことの一つです。

### [Retrofit2](http://square.github.io/retrofit/)

　RESTful API のクライアントをサクッと作れるライブラリ。これも Xamarin 用に移植してくれてる方がいます。

* [Refit by paulcbetts](http://paulcbetts.github.io/refit/) - Refit is a library heavily inspired by Square's Retrofit library

　こちらも、 Xamarin.iOS でも利用可能、PCL対応済み、カンペキです。

### [Picasso](http://square.github.io/picasso/)

　多機能且つ使いやすい Image Loader の Picasso。これは Xamarin の人が nuget パッケージを用意してくれています。

* [Square.Picasso](https://www.nuget.org/packages/Square.Picasso)

Picasso が依存している ``Square.OkHttp``, ``Square.OkIO`` も nuget パッケージが用意されていて、一緒に追加されます。

### [Android-Orma](https://github.com/gfx/Android-Orma)

　DroidKaigi当日には、作者 @gfx さんによる即席ランチセッションも聴けたORMライブラリ。若いライブラリなのでさすがに Xamarin版はありません。
　Xamarin.Android での ORMライブラリといえば [SQLite.NET](https://developer.xamarin.com/guides/cross-platform/application_fundamentals/data/part_3_using_sqlite_orm/) が有名ですが、使い方が面倒そうだったのと、このアプリのデータ構造と量で、リレーショナルDB使うこともないだろうと、 Key-Value Store である Akavache を使いました。これは以前 Qiita に書いたのでそちらを。

* [クロスプラットフォーム対応KVS Akavache を使ってお手軽にデータを保存する](http://qiita.com/amay077/items/356ad0028b7e6fbf089f)

　とはいえ、移植の際には、かなり強引な実装をしてしまいました。パフォーマンス悪いのは私の実装が原因です。

### [RxJava](https://github.com/ReactiveX/RxJava)

　これはもう説明不要でしょう。本家 Rx.NET を使用します。

* [Reactive-Extensions/Rx.NET: The Reactive Extensions for .NET](https://github.com/Reactive-Extensions/Rx.NET)

### [RxAndroid](https://github.com/ReactiveX/RxAndroid)

　これを使う最大の理由である ``AndroidSchedulers.mainThread()`` は、 Rx.NET では、 ``observable.ObserveOn(SynchronizationContext.Current)`` で代用できるので、不要でした。

### [ThreeTenABP](https://github.com/JakeWharton/ThreeTenABP)

　これも .NET の日付時刻系クラス(``DateTime``, ``DateTimeOffset``, ``TimeSpan``) で特に問題ありませんでした。しかし恥ずかしながらこのライブラリも知りませんで、Java では必須になりそうですね。

### [Stetho](http://facebook.github.io/stetho/)

　デバッグを強力に支援してくれるライブラリですね。これも知りませんでした。移植の時にはとりあえず関係なさそう、と思って代替品は探していません（汗

### [AndroidFlowLayout](https://github.com/LyndonChin/AndroidFlowLayout)

　View をいい感じに並べてくれるライブラリ。Xamarin.Android用の nuget パッケージがありました。

* [AndroidFlowLayout - NuGet Gallery](https://www.nuget.org/packages/AndroidFlowLayout/)

### Google Play services

　Map とか、Analytics とか。こちらも nuget に一通りパッケージが揃っています。Xamarin Components より優先的に使いましょう。

* [Xamarin Google Play Services - Maps - NuGet Gallery](https://www.nuget.org/packages/Xamarin.GooglePlayServices.Maps/)　
* [Xamarin Google Play Services - Analytics - Maps - NuGet Gallery](https://www.nuget.org/packages/Xamarin.GooglePlayServices.Analytics/)　

### [LikeButton](https://github.com/jd-alexander/LikeButton)

　Facebook の いいね!、Twitter の Fav! のようなボタンを提供してくれるライブラリ。押した時のアニメーションがイイ感じです。
　これの Xamarin 版は探してもなかったので、 LikeButton の ``.jar`` ファイルを入手して、自前で Java Binding Library プロジェクトを作って使用しています。

* [DroidKaigi2016Xamarin/LikeButton · amay077/DroidKaigi2016Xamarin](https://github.com/amay077/DroidKaigi2016Xamarin/tree/master/LikeButton)

　これを nuget に放流するのはやりたいことの2つ目。いくつかやったら [Xamarin から subscription もらえる](https://resources.xamarin.com/open-source-contributor.html)だろうか。。。

### [parceler](https://github.com/johncarl81/parceler)

　Parcel のことが大嫌いじゃなくなるライブラリ。移植に際しては、ModelクラスはPOCO(POJOの.NET版と思ってください)にしたかったので直接の代替品は探しませんでした。
　ModelクラスのParcel化はなんと [JSON.NET](http://www.newtonsoft.com/json) でJSONを介しちゃいました。悪手ですがパフォーマンスが気になる程でないならいいでしょ。

### [Crashlytics](https://try.crashlytics.com/)

　クラッシュレポート解析サービスですね。Xamarin なら [Xamarin Insights](https://xamarin.com/insights) がビルトインで使えるので、通常はそうするでしょう。Crashlytics 自体の Xamarin.Android 用ライブラリは、今のところ[存在しないみたい](https://twittercommunity.com/t/xamarin-and-fabric/37289/13)です。

### [multiline-collapsingtoolbar](https://github.com/opacapp/multiline-collapsingtoolbar)

　 Android Design Support Library の ``CollapsingToolbarLayout`` って、タイトルが複数行あると、展開しても表示されない(!)んですね。なんじゃそら！ってのを解決してくれるライブラリです。
　 Xamarin.Android向けのは探したけど見つかりませんでした。移植に際してクリティカルじゃなかったので、複数行にならない ``CollapsingToolbarLayout`` のままです。これも nuget パッケージ化したら需要あるかも。
 
### [CircularReveal](https://github.com/ozodrukh/CircularReveal)

　Lollipop で追加された CircularReveal アニメーションを、それ以前のOSでも行えるライブラリです。
　これもクリティカルでないので、Xamarin版には移植していません。

## まとめ

　DroidKaigi2016 の公式アプリは、ホストの @konifar さんはじめ、 [35名](https://github.com/konifar/droidkaigi2016/graphs/contributors) の精鋭有志の皆さんによる爆速開発で、 2/13 に v1.00 がリリース、イベント当日もアップデートされ、私も便利に利用させていただきました。
　
　一方、私の Xamarin.Android への移植は今やっと "とりあえず" 終わったばかり。
　しかも、移植の元にしたのが 2/10 付けのソースですが、その日から現在に至るまで本家にマージされた **Pull Request の数は 200超！** 。
　「これが若さか…。」これらの Xamarin版への移植はおじさんにはとても行う気が起きません。DroidKaigi2016公式アプリの Contributors の皆さんを尊敬します。
　
　が、ひとまず動くようになったので、ソースを公開します。モダンな Androidアプリを Xamarin.Android で実現する例としては有用だと思います。

* [amay077/DroidKaigi2016Xamarin: DroidKaigi2016 アプリをこっそりXamarinに移植](https://github.com/amay077/DroidKaigi2016Xamarin)

（できればこれを、 プラットフォーム非互換にできる箇所はPCLへ移動、各画面にViewModelを置いてMVVM化、Xamarin.iOS対応、Xamarin.Forms対応とか、いろいろと育てていきたいと思っているのですが、DroidKaigi参加直後で、あれもこれもやりたい病なので、実現は未定です。）
