---
layout: post
title: "マルチプラットフォーム MVVMフレームワーク「MvvmCross」を使う"
date: 2013-12-25 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#]
---
[Xamarin Advent Calendar 2013](http://qiita.com/advent-calendar/2013/xamarin) も最終日となりました。
最後は、「実用的な」マルチプラットフォーム開発のツールを紹介します。
<!--more-->
MvvmCross ってやつを紹介したいのですが、まずは事前知識からさらりと。

## MVVM パターンについて

MVVM は、 Model-View-ViewModel の頭文字を取ったものです。
MVC パターンの派生で、Microsoft が WPF/Silverlight のために作ったそうですが、今では JavaScript の [Knockout.js](http://knockoutjs.com/) などでも利用されています。

![](http://upload.wikimedia.org/wikipedia/commons/8/87/MVVMPattern.png)
(via [Model View ViewModel - WikiPedia](http://ja.wikipedia.org/wiki/Model_View_ViewModel) CC-BY 3.0) 

* Model - MVC の Model と一緒。ビジネスロジックはここに。
* View - WPF なら .xaml、iOS なら .storyboard、Android なら .xml それだけ。1行もコードを書かないのが理想。
	* DataBinding - ViewModel を監視して、ViewModel の情報を View に表示する。View でのユーザーからの入力を受け付けて ViewModel を変更したり、コマンドを実行する。
* ViewModel - View の為の Model。状態管理と View の為の情報＆機能公開、およびその為の Model の利用。”ロジック” はここにも書いちゃダメ。

いやもう絶対他のリソース読んだ方が分かりやすいですから(逃げ)。

* [MVVMパターンの常識 ― 「M」「V」「VM」の役割とは？ － ＠IT](http://www.atmarkit.co.jp/fdotnet/chushin/greatblogentry_02/greatblogentry_02_01.html)

## PCL(Portable Class Library) について

WPF, Silverlight, Windows Store App など、異なるプラットフォームでバイナリを共有できる “ポータブルな” クラスライブラリ。

* [ポータブル クラス ライブラリを使用して機能を共有する](http://msdn.microsoft.com/ja-jp/library/windowsphone/develop/jj714086(v=vs.105).aspx)

そして、先日、Xamarin.Android と iOS でもポータルクラスライブラリが使える/作れるようになりました。

* [PCL Projects and Visual Studio 2013 Support | Xamarin Blog](http://blog.xamarin.com/pcl-projects-and-vs2013/)

ポータブルクラスライブラリとして作られた DLL は、WPF でも Store App でも Android でも iOS でも使いまわせる、という事です。

## MvvmCross とは

さて本題。

MvvmCross は、様々なプラットフォームに対応した MVVMフレームワークです。対応プラットフォームをざっと挙げると、

* Xamarin.iOS
* Xamarin.Android
* Windows Phone
* Windows Store App
* WPF
* Mac

です。

MvvmCross を使うと、MVVM パターンでいうところの Model, ViewModel を複数のプラットフォームで共通にできます。

図にすると下のような感じです。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_01.png)

「Model でプラットフォーム固有の機能使いたい場合もあるじゃん？」とかにも対応しているので、これが全てでは無いですが、最初の説明としてはこんなもんです。

ホームページなどはこちら。
Evolve セッションの Slides の 1〜10ページ が分かりやすいですかね。

* [MvvmCross/MvvmCross - github](https://github.com/MvvmCross/MvvmCross)
* [Architecting Cross-Platform Apps with MvvmCross - Evolve 2013 Conference – Xamarin](http://xamarin.com/evolve/2013#session-dnoeeoarfj)

## Xamarin.Android Xamarin.iOS で、MvvmCross を使ったアプリを作ってみる

MvvmCross の現在 Stable なのは「v3」で、その Tutorials が

* [N plus 1 Videos Of MvvmCross · MvvmCross/MvvmCross Wiki](https://github.com/MvvmCross/MvvmCross/wiki/N-plus-1-Videos-Of-MvvmCross)

にあります。これがまたドットインストールも真っ青の充実ぶり。
この動画を順番に見ながら写経すれば使えるようになっちゃいます。

という訳で、最初の１つ「N=0」をトレースしてみましょう。

### 動画と違うところ

* Win でなく Mac、Visual Studio でなく Xamarin Studio を使います。Win+VS な人は動画をそのままトレースした方がよいでしょう。
* Nuget 使いません。Nuget から取得したらエラーになったので、マニュアルでアセンブリ追加します。Nuget で配信されるプロジェクトのスケルトンも使わずスクラッチで実装します。
* Windows Store App、Windows Phone は飛ばします。Mac なので。
* iOS での UI 作成について。動画ではコードでUI構築してますが、Storyboard を使いました。その影響で、少しコードが動画と変わっています。

では、開始〜。

### 1. PCL プロジェクトを作る

Xamarin Studio にて、新しいソリューションと「Portable Library」プロジェクトを作ります。プロジェクト名は “FirstDemo.Core”、ソリューション名は “FirstDemo” とします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_02.png)

### 2. MvvmCross のバイナリをダウンロードする

[MvvmCross/MvvmCross-Binaries の v3.1 branch](https://github.com/mvvmcross/MvvmCross-Binaries/tree/v3.1) をダウンロードなり Clone なりします。

### 3. プロジェクトにアセンブリ参照を追加する

FirstDemo.Core プロジェクトに、先ほどダウンロードした ``MvvmCross-Binaries-3.1/VS2012/bin/Release/Mvx/Portable`` の中の以下のファイルを参照追加します。

* Cirrious.CrossCore.dll
* Cirrious.MvvmCross.dll
* Cirrious.MvvmCross.Localization.dll

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_03.png)

### 4. FirstViewModel クラス、App クラスの実装

Nuget でインストールされるはずのクラスを実装します。

まず、ViewModels というフォルダを作ってその中に ``FirstViewModel`` クラスを作ります。

```csharp ViewModels/FirstViewModel.cs
using System;
using Cirrious.MvvmCross.ViewModels;

namespace FirstDemo.Core.ViewModels
{
    public class FirstViewModel : MvxViewModel
    {
        private string _firstName;
        public string FirstName
        {
            get { return _firstName; }
            set { _firstName = value; RaisePropertyChanged(() => FullName);}
        }

        private string _lastName;
        public string LastName
        {
            get { return _lastName; }
            set { _lastName = value; RaisePropertyChanged(() => FullName);}
        }

        public string FullName
        {
            get { return String.Format("{0} {1}", _firstName, _lastName); }
        }
    }
}
```

次に ``App`` クラスはルートに。

```csharp App.cs
using System;
using Cirrious.CrossCore.IoC;
using FirstDemo.Core.ViewModels;

namespace FirstDemo.Core
{
    public class App : Cirrious.MvvmCross.ViewModels.MvxApplication
    {
        public override void Initialize()
        {
            CreatableTypes()
                .EndingWith("Service")
                .AsInterfaces()
                .RegisterAsLazySingleton();

            RegisterAppStart<FirstViewModel>();
        }
    }
}
```

``MyClass.cs`` は使わないので削除します。

ここまでが[動画の 4:40](http://www.youtube.com/watch?v=_DHDMNB_IeY#t=281) くらいです。

### 5. Android の UI を作る

動画では [11:11](http://www.youtube.com/watch?v=_DHDMNB_IeY#t=671) から。

ソリューションに Android Application プロジェクトを追加します。名前は “FirstDemo.Droid” とします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_04.png) 

### 6. FirstDemo.Droid にアセンブリ参照を追加する

まず、FirstDemo.Core をプロジェクト参照で追加します。
次に、MvvmCross 関連のアセンブリ群、以下を追加します。動画では Nuget でやってるところです。

* Cirrious.CrossCore.dll
* Cirrious.CrossCore.Droid.dll *
* Cirrious.MvvmCross.dll
* Cirrious.MvvmCross.Droid.dll *
* Cirrious.MvvmCross.Binding.dll
* Cirrious.MvvmCross.Binding.Droid.dll *
* Cirrious.MvvmCross.Localization.dll

「*」付きのアセンブリは ``MvvmCross-Binaries-3.1/VS2012/bin/Release/Mvx/Droid/`` から、付いてないアセンブリは Core と同じく ``MvvmCross-Binaries-3.1/VS2012/bin/Release/Mvx/Portable/`` から追加します。後ろに “Droid” と付いているものは、プラットフォーム用のフォルダから持ってきましょうという事ですね。 

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_05.png)

### 7. FirstView クラス、Setup クラスの実装

こちらも Nuget が使えれば〜 のところ。

``FirstView`` は、MainView.cs をリファクタでリネーム＆ Views フォルダ移動して内容を下のように書き換えます。

```csharp Views/FirstView.cs
using System;
using Android.App;
using Android.Content;
using Android.Runtime;
using Android.Views;
using Android.Widget;
using Android.OS;
using Cirrious.MvvmCross.Droid.Views;

namespace FirstDemo.Droid.Views
{
    [Activity(Label = "FirstDemo.Droid", MainLauncher = true, Icon = "@drawable/icon")]
    public class FirstView : MvxActivity
    {
        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);
            SetContentView(Resource.Layout.Main);
        }
    }
}
```

Setup.cs はルートに作ります。

```csharp Setup.cs
using System;
using Cirrious.MvvmCross.Droid.Platform;
using Android.Content;
using Cirrious.MvvmCross.ViewModels;

namespace FirstDemo.Droid
{
    public class Setup : MvxAndroidSetup
    {
        public Setup(Context appContext) : base(appContext) { }

        protected override IMvxApplication CreateApp()
        {
            return new Core.App();
        }
    }
}
```

あ、 ``SplashScreen`` は省略します。説明が面倒なので。

### 8.  DataBinding の定義ファイルを作る

これも Nuget の代わりにやるところ。
データバインディングを行うために必要なファイルです。この手順を忘れて、ビルドエラーでしばらくハマってました。

``Resource/values`` フォルダ内に以下の xml ファイルを作成します。

```xml Resources/values/MvxBindingAttributes.xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
  <declare-styleable name="MvxBinding">
    <attr name="MvxBind" format="string"/>
    <attr name="MvxLang" format="string"/>
  </declare-styleable>
  <declare-styleable name="MvxControl">
    <attr name="MvxTemplate" format="string"/>
  </declare-styleable>
  <declare-styleable name="MvxListView">
    <attr name="MvxItemTemplate" format="string"/>
    <attr name="MvxDropDownItemTemplate" format="string"/>
  </declare-styleable>
  <item type="id" name="MvxBindingTagUnique"/>
  <declare-styleable name="MvxImageView">
    <attr name="MvxSource" format="string"/>
  </declare-styleable>
</resources>
```

### 9. 画面を作る

画面のレイアウトを作ります。
Xamarin Studio の Android用 UI デザイナは強力なので、ここだけは Visual Studio に勝っていると言えます。

``Main.axml`` を開いて、レイアウトされているボタンを削除し、「Plain Text」を2つと、Text(Mid) を1つ、縦に並べて配置します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_06.png)

### 10. データバインディングを記述する

``Main.axml`` の「ソース」を開いて、データバインディングについての記述をします。完成形は下のようになります。

```xml Resources/layout/Main.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:local="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        local:MvxBind="Text FirstName" />
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        local:MvxBind="Text LastName" />
    <TextView
        android:text="Medium Text"
        android:textAppearance="?android:attr/textAppearanceMedium"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        local:MvxBind="Text FullName" />
</LinearLayout>
```

これで Android UI の実装は終わり。動画では [16:20](http://www.youtube.com/watch?v=_DHDMNB_IeY#t=980) あたり。
Android 側では、初期化を除けば **何も実装してない** のがミソ。

### 11. Android アプリを動かしてみる

エミュレータで実行してみると、こんな感じです。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_07.gif)

Steve を痛恨のスペルミスしたけど、まあいいや。

では続いて、iOS アプリの実装です。

### 12. iOS の UI を作る

動画では [25:58](http://www.youtube.com/watch?v=_DHDMNB_IeY#t=1558) から。

ソリューションに iPhone Application プロジェクトを追加します。Storyboard を使ってみましょう。名前は “FirstDemo.Touch” とします。(iOS 版の接尾辞に ”Touch” を使うのは、Xamarin.iOS の以前の名称である ”MonoTouch” からだと思いますが、大文字で始まる .NET 文化の中では “iOS” はとかく都合が悪いので、Touch という接尾辞は私も気に入っています。)

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_08.png)

### 13. FirstDemo.Touch にアセンブリ参照を追加する

Android の場合と同様、FirstDemo.Core をプロジェクト参照で追加します。
次に、MvvmCross 関連のアセンブリ群、以下を追加します。

* Cirrious.CrossCore.dll
* Cirrious.CrossCore.Touch.dll *
* Cirrious.MvvmCross.dll
* Cirrious.MvvmCross.Touch.dll *
* Cirrious.MvvmCross.Binding.dll
* Cirrious.MvvmCross.Binding.Touch.dll *
* Cirrious.MvvmCross.Localization.dll

「*」付きのアセンブリは MvvmCross-Binaries-3.1/VS2012/bin/Release/Mvx/Touch/ から、付いてないアセンブリは Core と同じく MvvmCross-Binaries-3.1/VS2012/bin/Release/Mvx/Portable/ から追加します。後ろに “Touch” と付いているものは、プラットフォーム用のフォルダから持ってきましょうという事ですね。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_09.png)

### 14. Setup クラス, AppDelegate クラス、FirstView クラスの実装

とここまで書いておいて、MvvmCross は実は Storyboard で使う時は少し細工が必要な事に気づいた。。。

MvvmCross の作者である slodge さんが下で回答されています。

* [monotouch - MvvmCross and Xcode Storyboard - Stack Overflow](http://stackoverflow.com/a/16323115)

という訳でここからのコードは、上記で示されている ‘eh’ もミックスしたもので、動画とは少し異なります。動作は同じです。

まず、Setup.cs をルートに作成して以下のように実装します。

```objc Setup.cs
using System;
using Cirrious.MvvmCross.Touch.Platform;
using Cirrious.MvvmCross.Touch.Views.Presenters;
using Cirrious.MvvmCross.ViewModels;

namespace FirstDemo.Touch
{
    public class Setup : MvxTouchSetup
    {
        public Setup(MvxApplicationDelegate appDelegate, IMvxTouchViewPresenter presenter)
            : base(appDelegate, presenter) { }

        protected override IMvxApplication CreateApp()
        {
            return new Core.App();
        }
    }
}
```

既存の AppDelegate.cs を以下のように書き換えます。

```objc AppDelegate.cs
using MonoTouch.Foundation;
using Cirrious.MvvmCross.Touch.Platform;
using MonoTouch.UIKit;
using Cirrious.MvvmCross.Touch.Views.Presenters;

namespace FirstDemo.Touch
{
    [Register("AppDelegate")]
    public partial class AppDelegate : MvxApplicationDelegate
    {
        public override UIWindow Window { get; set; }

        public override void FinishedLaunching(UIApplication application)
        {
            var setup = new Setup(this, new MvxTouchViewPresenter(this, Window));
            setup.Initialize();
        }
    }
}
```

FirstView.cs は、FirstDemo.FirstDemoViewController.cs をリネームして作成します。Views フォルダを作って移動もしましょう。
また、``MvxViewConroller`` から派生させるように変更します。

 ![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_10.png)

### 15. iOS の UI を作る

iOS 版の画面をレイアウトします。
動画ではコードで ``UITextField`` などを配置していますが、 **UI をコードで記述する事は万死に値する** ので、Xamarin Studio の iOS デザイナもしくは Xcode の Interface Builder を使います。

下の図は、Xamarin Studio α版の iOS デザイナ を使った例です。[以前に紹介した](http://qiita.com/amay077/items/716742474bce343c5729)ものです。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_11.png)

TextField 2つと、Label を、``textEditFirst``, ``textEditLast``, ``labelFull`` という変数にしておきます。

### 16. データバインディングを記述する

MvvmCross を iOS で使う場合、残念ながら storyboard 側にバインディングを記述する事はできないので、``FirstView.cs``  にコードで記述します。

``ViewDidLoad`` に以下のように追記します。

```objc Views/FirstView.cs
using System;
using Cirrious.MvvmCross.Binding.BindingContext;
using Cirrious.MvvmCross.Touch.Views;
using Cirrious.MvvmCross.ViewModels;
using FirstDemo.Core.ViewModels;

namespace FirstDemo.Touch
{
    public partial class FirstView : MvxViewController
    {
        /* 省略 */

        public override void ViewDidLoad()
        {
            this.Request = new MvxViewModelRequest<FirstViewModel>(
                null, null, new MvxRequestedBy());           
            base.ViewDidLoad();

            var set = this.CreateBindingSet<FirstView, FirstViewModel>();
            set.Bind(textEditFirst).To(vm => vm.FirstName);
            set.Bind(textEditLast).To(vm => vm.LastName);
            set.Bind(labelFull).To(vm => vm.FullName);
            set.Apply();
        }

        /* 省略 */
    }
}
```

これで iOS 側も実装終了、動画では [33:40](http://www.youtube.com/watch?v=_DHDMNB_IeY#t=2020) まで来ました。

### 17. iOS アプリを動かしてみる

動画では、Windows であるため Mac にリモート接続して実行していますが、Mac+Xamarin Studio なら即実行できます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_mvvmcross_2_12.gif)

## MvvmCross についてのまとめ

今日の完成版のコードは

* [NPlus1DaysOfMvvmCrossWithXamarinStudio / N-00-FirstDemo - github](https://github.com/amay077/NPlus1DaysOfMvvmCrossWithXamarinStudio/tree/master/N-00-FirstDemo)

に置いておきました。

ちょっと STEP が多くなっちゃいましたが、MvvmCross を使うと、複数のプラットフォームで ViewModel-Model を共通化できることが分かったと思います。

[N+1 Days-](https://github.com/MvvmCross/MvvmCross/wiki/N-plus-1-Videos-Of-MvvmCross) は、現在 39(!!) まであります。動画観てるだけでもわかった感じになります。（私は N=8 まで観た気がします。）

また、 @MvvmCross にツイートすると、気さくに（英語ですが）回答してくれます。

使いこなせば強力な武器になる MvvmCross 、今後も要チェックです。

## Xamarin Advent Calendar 2013 まとめ

勢いに任せて作った XAC2013、なんとか完走できました。
「全部俺」でもいいやと思っていましたが、私含め5名の方に参加して頂けました、ありがとうございました。

[日経ソフトウェア](http://www.amazon.co.jp/gp/product/B00H2SBO4E?ie=UTF8&camp=1207&creative=8411&creativeASIN=B00H2SBO4E&linkCode=shr&tag=oku2008-22&=books&qid=1387973043&sr=1-1&keywords=%E6%97%A5%E7%B5%8C%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A8%E3%82%A2+2014%E5%B9%B42%E6%9C%88%E5%8F%B7) によると、「2014年にブレークする技術」に Xamarin はありませんでしたが、MS と提携以後のプッシュぶりを見ていると、今後、来年のブレークに期待できそうです。

昨日の @atsushieno さんの [Xamarin 創立からの苦労話](http://atsushieno.hatenablog.com/entry/2013/12/24/213950) を知ると、安易に「ライセンス高ぇ！値下げPlz！」などとは言えませんが、ユーザ(デベロッパ)を増やしたいのもまた事実。α/β版だけでも 30days のトライアル期限なくしてもらえたいら嬉しいですね。

Xamarin Advent Calendar 2013 はこれで終わりですが、今後もちまちまと Tips みたいなものを書いてくつもりですので、 [Qiita のタグ](http://qiita.com/tags/xamarin) をチェックしてもらえると嬉しいです。

そして Xamarin Advent Calendar 2014 でまたお会いしましょう。

