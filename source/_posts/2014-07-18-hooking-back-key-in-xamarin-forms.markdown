---
layout: post
title: "Xamarin.Forms、Android での BACK キーの制御"
date: 2014-06-17 01:00
comments: true
categories: [Xamarin, C#, Android]
---
[Xamarin.Forms でどうにかしたい iOS と Android の違い](http://qiita.com/amay077/items/12979585ac3e2dcacacb) の「BACKキーの制御」の **現時点(1.1.0.6201)** での回答。
<!--more-->

Android の BACKキーの制御を、Xamarin.Forms ではどう扱えるかを調べた。

## シナリオ

Xamarin.Forms による画面１(MainPage)、２(SecondPage)があり、MainPage では BACKキーで戻る(=アプリ終了)事ができるが、SecondPage ではBACKキーが効かない、ようにしたい。


## 対策

まず画面１と２はこんな感じ。ボタンを押したら画面２へ遷移するだけ。

```csharp Pages.cs
// 画面１
public class MainPage : ContentPage
{
    public MainPage() 
    {
        var button = new Button
        {
            Text = "To Second",
            VerticalOptions = LayoutOptions.Center,
        };

        button.Clicked += (sender, e) => 
        {
            this.Navigation.PushAsync(new SecondPage());
        };

        Content = button;
    }
}

// 画面２
public class SecondPage : ContentPage
{
    public SecondPage()
    {
        Content = new Label
        {
            Text = "Second"
        };
    }
}
```

ここからが本題。
まず Android側のエントリポイントである ``MainActivity.cs`` は以下のように、``ContentPage`` プロパティを設ける。そして ``OnBackPressed`` メソッドを override して、MainPage だったら OnBackPressed を親へ伝搬する。

```csharp MainActivity.cs
[Activity(Label = "ScrollTest.Android.Android", MainLauncher = true)]
public class MainActivity : AndroidActivity
{
    protected override void OnCreate(Bundle bundle)
    {
        base.OnCreate(bundle);

        Xamarin.Forms.Forms.Init(this, bundle);

        SetPage(new NavigationPage(new MainPage()));
    }

    internal Page ContentPage
    {
        get;
        set;
    }

    public override void OnBackPressed()
    {
        if (this.ContentPage is MainPage)
        {
            base.OnBackPressed();
        }
    }
}
```

次に、MainActivity.ContentPage への設定を行うコードは以下の通り。
PageRenderer を拡張して ExportRenderer することで、すべての Page にフックをかけ、Page の表示時に MainActivity.ContentPage に設定する。

```csharp MyPageRenderer.cs
using System;
using Xamarin.Forms.Platform.Android;
using Android.App;
using Xamarin.Forms;
using ScrollTest.Android;
using Android.Views;
using Android.Graphics;

[assembly:ExportRenderer(typeof(ContentPage), typeof(MyPageRenderer))]

namespace ScrollTest.Android
{
    public class MyPageRenderer : PageRenderer
    {
        protected override void OnElementChanged(ElementChangedEventArgs<Xamarin.Forms.Page> e)
        {
            base.OnElementChanged(e);

            // なんとなく不安なので weak にしてみた
            var activity = new WeakReference<MainActivity>(this.Context as MainActivity);

            e.NewElement.Appearing += (_, __) =>
            {
                MainActivity a;
                if (activity.TryGetTarget(out a)) {
                    a.ContentPage = e.NewElement;    
                }
            };
        }
    }
}
```

これで、画面１(MainPage)の時だけ BACKキーが効くようにできる。


### ``Appearing`` イベントが必要なの？

　Xamarin.Forms の Android実装では、画面遷移の度に **「同じインスタンスの MainActivity」** が使いまわされる、さらに ``OnElementChanged`` は、各Pageにつき１度しか発生しない。その為、画面１→２→１と遷移すると ``MainActivity.ContentPage`` は ``SecondPage`` のままになってしまう。ので ``Appearing`` イベントで表示の度に MainActivity.ContentPage を設定する必要がある。

### ``AndroidActivity`` に static な ``BackPressed`` イベントがあるんだけど…

イベントハンドラの定義は 
``public delegate bool BackButtonPressedEventHandler(object sender, EventArgs e);``
となっていて、``true`` を返すと BACK キーを無効にできるようなのだけど、``sender`` は ``MainActivity``だし、``EventArgs`` は Page を取得できないしで使えないじゃん。。。

なんだかすごく発展途上な気がする、その内いろいろ整備されそうなので、それまで待った方が良い気がします。。。

