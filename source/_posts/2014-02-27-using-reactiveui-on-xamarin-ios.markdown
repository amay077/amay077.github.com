---
layout: post
title: "MVVMフレームワーク「ReactiveUI」を Xamarin.iOS で使ってみる"
date: 2014-02-27 21:34
comments: true
categories: [Xamarin, Android, MVVM, C#, iOS]
---
ReactiveUI は、Reactive Extensions を全面的に取り入れた クロスプラットフォームな MVVMフレームワークです。
<!--more-->
* [ReactiveUI](http://www.reactiveui.net/)

作者は GitHub の中の人 [Paul Betts](http://twitter.com/xpaulbettsx) 氏、[Xamarin の MVP](http://xamarin.com/mvp) でもあります。

元々は WPF, Silverlight, WinRT, Windows Phone に対応していましたが、Xamarin.iOS や Xamarin.Android, Xamarin.Mac にも [対応が進んで来た](http://blog.paulbetts.org/index.php/2013/03/12/reactiveui-4-5-is-released/) ので、使ってみる事にしました。

Visual Studio + WPF 等なら、nuget から取得できて楽なんでしょうけども、なにせ Mac なので、Xamarin Studio のみでいきます。

## とりあえず使ってみる

Github が公開した [GitHub's Xamarin starter apps](http://log.paulbetts.org/open-source-githubs-xamarin-starter-apps/), これに ReactiveUI も含まれているので、こちらを Clone して Xamarin Studio で開いてビルド、すぐ動きます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_reactiveui_01.png)

これ、ViewModel側で UUID を生成して、View側の Label にバインドしているのですが、何ともシンプル過ぎて…。

それでもこのフレームワークの構成を知るには十分です。

## ReactiveUI に必要なもの

ソリューションツリーを見ると次の4つのプロジェクトがあります。

* Starter-Core-Android
* Starter-Core-iOS
* Starter-Android
* Starter-iOS

### ViewModel-Model層

Starter-Core-xxx は、ディレクトリ的には同じ場所にあり、Android用とiOS用のプロジェクトファイル(.csproj)が用意してあるだけです。ここはアプリケーションの ViewModel-Model層になります。PCL化はされていないようですね(その内、とサイトに書いてありました)。

サンプルで用意されてる ViewModel を見てみます。

```csharp TestViewModel.cs
using System;
using ReactiveUI;
using System.Runtime.Serialization;

namespace Starter.Core.ViewModels
{
    [DataContract]
    public class TestViewModel : ReactiveObject
    {
        string _TheGuid;
        [DataMember] public string TheGuid {
            get { return _TheGuid; }
            set { this.RaiseAndSetIfChanged(ref _TheGuid, value); }
        }

        public TestViewModel()
        {
            TheGuid = Guid.NewGuid().ToString();
        }
    }
}
```

MvvmCross とか、他の MVVM-FW とだいたい同じですね(そりゃそうだ)。
基底クラスの ``ReactiveObject`` が、BaseViewModel的な役割をします。(が、Reactive を冠しているだけに、随所で Rx の力が発揮される、はずです←まだ分かってない)

このコードでは、TestViewModel の生成と同時に、Guid を生成して、``TheGuid`` プロパティに設定しています。

### View層

Starter-Android, Starter-iOS はそれぞれの View層になります。

Starter-iOS の TestViewController.cs を見てみます。

```csharp TestViewController.cs
namespace Starter.Views
{
    public partial class TestViewController : ReactiveViewController, IViewFor<TestViewModel>
    {
		[省略]

        public override async void ViewDidLoad()
        {
            base.ViewDidLoad();
            this.OneWayBind(ViewModel, vm => vm.TheGuid, v => v.TheGuid.Text);

            ViewModel = await BlobCache.LocalMachine.GetOrCreateObject("TestViewModel", () => {
                return new TestViewModel();
            });
        }

        TestViewModel _ViewModel;
        public TestViewModel ViewModel {
            get { return _ViewModel; }
            set { this.RaiseAndSetIfChanged(ref _ViewModel, value); }
        }

		[省略]
    }
}
```

``UIViewController`` ではなく ``ReactiveViewController`` から派生させてます。この辺もよくあるやり方。``IViewFor`` は、今はスルーで。

バインドは ``this.OneWayBind`` で。
ViewModel の TheGuid プロパティを、View の TheGuidラベルの Text プロパティへ単方向(OneWay)バインドしてます。

TestViewModel の生成は、ここでは Akavache というストレージライブラリの生成を待ってから行っていますが、Akavache を使わない場合は普通に ``this.ViewModel = new TestViewModel()`` で OK でしょう。

これで、TestViewModelの生成 → Guidの生成 → vm.TheGuidプロパティへ設定 → vm より TheGuid の変更が通知される → View側のBindingが変更を検知 → Viewのラベルを書き換える、という流れになります。

## ちょっと拡張してみる

### 双方向バインディング

ViewModel→View だけでなく、View→ViewModel もやってみましょう。

まず TestViewModel にプロパティを追加します。
プロパティは ``MyName`` とします。
初期値として "Enter your name" とでも設定しましょうか。 

```csharp TestViewModel.cs
namespace Starter.Core.ViewModels
{
    [DataContract]
    public class TestViewModel : ReactiveObject
    {
        string _TheGuid;
        [DataMember] public string TheGuid {
            get { return _TheGuid; }
            set { this.RaiseAndSetIfChanged(ref _TheGuid, value); }
        }

        string _myName;
        [DataMember] public string MyName {
            get { return _myName; }
            set { this.RaiseAndSetIfChanged(ref _myName, value); }
        }

        public TestViewModel()
        {
            TheGuid = Guid.NewGuid().ToString();
            
            this.MyName = "Enter your name";
        }
    }
}
```

次に Interface Builder で TestViewController に、UITextField と UILabel を追加し、Outlet を "MyText", "MyLabel" とします。これで Xamarin.iOS から ``MyText``, ``MyLabel`` でインスタンスにアクセスできるはず、ですよね。

``MyText``, ``MyLabel`` に、vm.MyName をバインドします。

```csharp TestViewController.cs
namespace Starter.Views
{
    public partial class TestViewController : ReactiveViewController, IViewFor<TestViewModel>
    {
		[省略]

        public override async void ViewDidLoad()
        {
            base.ViewDidLoad();
            this.OneWayBind(ViewModel, vm => vm.TheGuid, v => v.TheGuid.Text);

            this.Bind(ViewModel, vm=> vm.MyName, v => v.MyText.Text);
            this.OneWayBind(ViewModel, vm => vm.MyName, v => v.MyLabel.Text);

            ViewModel = await BlobCache.LocalMachine.GetOrCreateObject("TestViewModel", () => {
                return new TestViewModel();
            });
        }

		[省略]
    }
}
```

編集できる ``MyText`` は ``this.Bind`` を使って双方向バインドします。プロパティの値を表示するだけの ``MyLabel`` は、 ``this.OneWayBind`` で。

これで動かしてみます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_reactiveui_02.gif)

UITextField への入力が、vm.MyName へ適用され、その変更を MyLabel に表示させる、という流れです。

今日はこの辺で。まだ全然 Reactive じゃないですが、次回以降、Command の実装やバインディングについて試してみようと思います。

ここまでのコードは、

* https://github.com/amay077/starter-mobile/tree/N_plus_1

に置いておきます。徐々に進化させていこうと思います。