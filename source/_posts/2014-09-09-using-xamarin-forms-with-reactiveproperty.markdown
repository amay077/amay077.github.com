---
layout: post
title: "Xamarin.Forms と ReactiveProperty で快適MVVM生活"
date: 2014-09-09 21:38:29 +0900
comments: true
categories: [Xamarin, Android, iOS, Reactiveextensions, ReactiveProperty]
---
　[Xamarin.Forms](http://www.buildinsider.net/mobile/xamarintips/0005) は、Xamarin に新たに搭載されたクロスプラットフォームUIフレームワーク＆MVVMフレームワークです。
<!--more-->
　[ReactiveProperty](http://okazuki.hatenablog.com/entry/2014/05/07/014133) は、MVVMの(特に ViewModelの)実装を強力にサポートしてくれる、[Reactive Extensions](http://www.atmarkit.co.jp/fdotnet/introrx/index/) を基盤としたライブラリです。

 両者を組み合わせると、Android/iOSアプリが COOL な感じで書けるんじゃないか、という事で試してみました。

## 0. 環境など

Mac + Xamarin Studio を使いますが、Windows + Visual Studio + Xamarin-Addin でもイケると思います。

## 1. 導入

### プロジェクトの作成

新規ソリューションを、［C#］−［Mobile Apps］−［Blank App(Xamarin.Forms Portable)］で作成します。

### PCL の Profile を変更

　作成されたソリューションの一番上にあるプロジェクト(.Android とか .iOS が付いていないやつ)のプロジェクト設定を開いて Profile を **PCL 4.5 - Profile49** に変更します。元々の Profile78 では ReactiveProperty が Nuget からインストールできないためです。最近のプラットフォームを対象にするなら、あまり影響はなさそうです。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_forms_with_reactiveproperty_01.png)

### Nuget で Reactive Extensions と ReactiveProperty を追加

　メニューの［プロジェクト］ー［Add Packages］で Nuget のダイアログを開き、図のように 「Reactive Extensions - Main Library」と「ReactiveProperty Portable」を追加します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_forms_with_reactiveproperty_02.png)

（Reactive Extensions の追加の際、なにやらWarningが出るようですが、とりあえず進めます。）

## 2. ViewModel の実装

　PCL のプロジェクトに、``FirstViewModel.cs`` を作成します。
　``FirstViewModel`` は、以下のようなプロパティとコマンドを持ちます。

* InputTextプロパティ : EditBox の入力に応じて更新
* DisplayTextプロパティ : InputText の変化から1秒後に、InputText を大文字にして更新
* Clearコマンド : InputText が 'clear' の時のみ有効。実行すると InputText を空にする。

これらの実装が下のようになります。

```csharp FirstViewModel.cs
using System;
using Codeplex.Reactive;
using System.Reactive.Linq;

namespace FormsWithRxProperty.ViewModels
{
    public class FirstViewModel
    {
        private readonly ReactiveProperty<string> _inputText = 
            new ReactiveProperty<string>("Hoge");
        public ReactiveProperty<string> InputText 
        { 
            get { return _inputText; }
        }

        public ReactiveProperty<string> DisplayText
        {
            get; private set;
        }

        public ReactiveCommand Clear
        {
            get; private set;
        }

        public FirstViewModel()
        {
            // DisplayText は、InputText の変更から1秒後に大文字にして更新
            this.DisplayText = _inputText
                .Delay(TimeSpan.FromSeconds(1))
                .Select(x => x.ToUpper())
                .ToReactiveProperty();

            // InputText が `clear` の時に実装可能
            this.Clear = _inputText
                .Select(x => x.Equals("clear"))
                .ToReactiveCommand();
            // 実行されたら、InputText を空にする
            this.Clear.Subscribe(_ => _inputText.Value = String.Empty);
        }
        
    }
}
```

　面倒な ``INotifyPropertyChanged`` の実装が必要なく、すっきりと記述できます。
　また、他のプロパティに関連して(反応して)値が変化するプロパティや、コマンドの利用可否などが、Reactive Extensions の機能により、流れるように記述できます。

## 3. 画面及び ViewModel との Binding の実装

　画面(UI)は、Xamarin.Forms の恩恵で、Android/iOS 共通で実装できます。XAML も使えますが、よく知らないのでコードでUIを記述します。

　PCL のプロジェクトに、 ``FirstPage.cs`` を作成し、以下のように実装します。

```csharp FirstPage.cs
using System;
using Xamarin.Forms;
using FormsWithRxProperty.ViewModels;

namespace FormsWithRxProperty.Pages
{
    public class FirstPage : ContentPage
    {
        public FirstPage()
        {
            // UI
            var entry = new Entry
            {
                Text = "Hello, Forms!",
                VerticalOptions = LayoutOptions.Center,
                HorizontalOptions = LayoutOptions.FillAndExpand,
            };

            var label = new Label
            {
                VerticalOptions = LayoutOptions.Center,
                HorizontalOptions = LayoutOptions.CenterAndExpand,
            };

            var button = new Button
            {
                Text = "Clear (type 'clear' to enable)",
                VerticalOptions = LayoutOptions.Center,
                HorizontalOptions = LayoutOptions.FillAndExpand,
            };

            this.Content = new StackLayout
            {
                Padding = new Thickness(50f),
                VerticalOptions = LayoutOptions.Start,
                HorizontalOptions = LayoutOptions.Fill,
                Orientation = StackOrientation.Vertical,
                Children =
                {
                    entry,
                    label,
                    button
                }
            };

            // ViewModel との Binding
            this.BindingContext = new FirstViewModel();
            entry.SetBinding<FirstViewModel>(Entry.TextProperty, vm=>vm.InputText.Value);
            label.SetBinding<FirstViewModel>(Label.TextProperty, vm=>vm.DisplayText.Value);
            button.SetBinding<FirstViewModel>(Button.CommandProperty, vm=>vm.Clear);
        }
    }
}
```

　ちょっと長いですが、画面に「エディットボックス」「ラベル」「ボタン」が縦に並んでいるだけです。

　下部の４行で、``FirstViewModel`` の各プロパティ、コマンドと Bind しています。

　もともとあった ``App.cs`` は、``FirstPage`` を生成するだけにします。

```csharp App.cs
using System;
using Xamarin.Forms;
using FormsWithRxProperty.Pages;

namespace FormsWithRxProperty
{
    public class App
    {
        public static Page GetMainPage()
        {	
            return new FirstPage();
        }
    }
}
```


## 動かす！

 .Android か .iOS の付いたプロジェクトをスタートアップにして、実行します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_forms_with_reactiveproperty_03.gif)

### 追記 2014.9.10

実機で動作確認するの忘れてました（実機はAOTなのに対してiOSシミュレータはJITなのでリフレクションとかが普通に動いてしまう）。
実機でも問題なく動作しました！

## まとめ

　Reactive Extensions のメリットを活かして MVVM を構築できる ReactiveProperty と、ワンソースで Android/iOS の画面を定義でき、さらに Binding までも共通にできる Xamarin.Forms の組み合わせは、今後のモバイルアプリケーション開発をとても効率的にしてくれます、 **そしてなにより楽しい！**

　今回のサンプルプログラムは

* [amay077/XamarinFormsWithReactivePropertySample](https://github.com/amay077/XamarinFormsWithReactivePropertySample/tree/master)

　に置きましたので、是非試してみてください。


### ReactiveProperty

* [ReactiveProperty オーバービュー - かずきのBlog@hatena](http://okazuki.hatenablog.com/entry/2014/05/07/014133)
* [ReactiveProperty - MVVM Extensions for Rx - Download: ReactiveProperty v1.0](https://reactiveproperty.codeplex.com/releases/view/132232)
* [neue cc - ReactiveProperty : Rx + MVVMへの試み](http://neue.cc/2011/08/26_341.html)
* [ReactivePropertyを使いたい人のための、ReactiveExtensions入門（その１） | 泥庭](http://yone64.wordpress.com/2014/06/20/reactiveproperty%E3%82%92%E4%BD%BF%E3%81%84%E3%81%9F%E3%81%84%E4%BA%BA%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AE%E3%80%81reactiveextensions%E5%85%A5%E9%96%80%EF%BC%88%E3%81%9D%E3%81%AE%EF%BC%91%EF%BC%89/)

### Xamarin.Forms

* [Xamarin.Forms | Xamarin](http://developer.xamarin.com/guides/cross-platform/xamarin-forms/)
* [Xamarin.Formsの基本構想と仕組み - Build Insider](http://www.buildinsider.net/mobile/insidexamarin/14)
* [Xamarin.Forms - Build Insider](http://www.buildinsider.net/tagcloud?tag=Xamarin.Forms)
* [Xamarin.Forms ListViewでTwitter風のレイアウトを作成してみました（機種依存コードなし） - SIN@SAPPOROWORKSの覚書](http://furuya02.hatenablog.com/entry/2014/08/08/003036)
