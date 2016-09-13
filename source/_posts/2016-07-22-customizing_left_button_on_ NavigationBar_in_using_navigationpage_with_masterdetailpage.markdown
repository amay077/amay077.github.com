---
layout: post
title: "NavigationPage + MasterDetailPage の時に iOS の NavigationBar の左ボタンをカスタマイズする"
date: 2016-07-22  23:59:59 +0900
comments: true
categories: [Xamarin, Xamarin.Forms, iOS, C#]
---

Xamarin.Forms では、左からスライドして出てくるメニューを持つ画面を ``MasterDetailPage`` で作成します。

<!--more-->

* [Master-Detail Page - Xamarin](https://developer.xamarin.com/guides/xamarin-forms/user-interface/navigation/master-detail-page/)

一方、普通に画面遷移していく場合は ContentPage などを ``NavigationPage`` でラップしてあげます。

* [Hierarchical Navigation - Xamarin](https://developer.xamarin.com/guides/xamarin-forms/user-interface/navigation/hierarchical/)

## やりたいこと

何がしたいかというと、両者を組み合わせたいんです。こういうことってよくありませんかね？

![](https://dl.dropboxusercontent.com/u/264530/qiita/customizing_left_navvigationbar_button_in_masterdetail_with_navigation_pages_01.png)

起動画面で「新規ユーザー登録」があって、「ユーザー登録画面」を経て、メインの画面に遷移する、メイン画面にはスライドメニューがある、というパターン。これを Xamarin.Forms でやりたいのです。

## 問題

ところが、 ``NavigationPage`` で遷移していく画面の中に ``MasterDetailPage`` があると、 ``NavigationPage`` の方が勝ってしまい、ナビゲーションバーには「BACK」ボタンが表示されてしまいます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/customizing_left_navvigationbar_button_in_masterdetail_with_navigation_pages_02.png)

これを消そうと、``MasterDetailPage`` のコンストラクタで ``NavigationPage.SetHasBackButton(this, false)`` してみます。

その結果がこれ。

![](https://dl.dropboxusercontent.com/u/264530/qiita/customizing_left_navvigationbar_button_in_masterdetail_with_navigation_pages_03.png)

Android の方は望む結果になったけど、iOSの方はうーん…、BACKボタンは消えたけど、メニューを表示させるボタンが出ません。

しょうがないので、iOS の場合だけ、ナビゲーションバーの左ボタンをどうにかして追加してみます。

## 私が求めていたソリューション（CustomRenderer編）

Xamarin.Forms のお供、CustomRenderer です。

* [Customizing Controls on Each Platform - Xamarin](https://developer.xamarin.com/guides/xamarin-forms/custom-renderer/)
* [【Xamarin.Forms】ViewRendererと仲良くなるための簡易チュートリアル - ぴーさんログ](http://ticktack.hatenablog.jp/entry/2016/06/11/124751)

``MasterDetailPage`` の iOS向けCustomRenderer を作って、ネイティブ側でナビゲーションバーをカスタマイズしてみます。

```csharp CustomMasterDetailRenderer.cs
using System;
using MasterDetail.iOS;
using MonoTouch.UIKit;
using Xamarin.Forms;
using Xamarin.Forms.Platform.iOS;

[assembly: ExportRenderer(typeof(MasterDetailPage), typeof(CustomMasterDetailRenderer))]
namespace MasterDetail.iOS
{
    public class CustomMasterDetailRenderer : PhoneMasterDetailRenderer
    {
        public override void ViewWillAppear(bool animated)
        {
            base.ViewWillAppear(animated);

            var page = Element as MasterDetailPage;

            var navigationItem = this.NavigationController.TopViewController.NavigationItem;
            navigationItem.LeftBarButtonItems = new UIBarButtonItem[]
            {
                new UIBarButtonItem("MENU", UIBarButtonItemStyle.Plain, (_, __) => 
                { 
                    page.IsPresented = !page.IsPresented; 
                })
            };
        }
    }
}
```
「MENU」ってボタンを、ナビゲーションバーの左側に追加しています。
こんな CustomRenderer を iOS 側のプロジェクトに追加して実行してみます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/customizing_left_navvigationbar_button_in_masterdetail_with_navigation_pages_04.png)

オーケーオーケー、これが私が求めていたソリューションです。

## 私が求めていたソリューション（Effects編）

が、CustomRenderer にはいくつか考えなければならないことがあります。

* 上で作った ``CustomMasterDetailRenderer.cs`` は ``PhoneMasterDetailRenderer`` というクラスを継承しています。が、実はこれは iPhone 用で、実はタブレット（iPad）用に ``TabletMasterDetailRenderer`` というクラスもあります。これの CustomRenderer も用意しなければなりませんか？
* CustomRenderer はベースとなる ViewRenderer を「継承」して作ります。そして C# は多重継承を許していません、この意味が分かるな？別の機能を拡張したいと思ったら``CustomMasterDetailRenderer``から派生させるしかなくなります。

で、 Xamarin.Forms には、v2.1 から既存機能の拡張に Effects という選択肢が加わりました。

* [Customizing Controls with Effects - Xamarin](https://developer.xamarin.com/guides/xamarin-forms/effects/)
* [【Xamarin.Forms 2.1.0(プレビュー)】Effects - ぴーさんログ](http://ticktack.hatenablog.jp/entry/2016/01/26/020248)

では、``CustomMasterDetailRenderer.cs`` を Effects に変えてみましょう。

```csharp CustomMasterDetailEffect.cs
using System;
using MasterDetail.iOS;
using MonoTouch.UIKit;
using Xamarin.Forms;
using Xamarin.Forms.Platform.iOS;

[assembly: ResolutionGroupName("mycompany")]
[assembly: ExportEffect(typeof(CustomMasterDetailEffect), "CustomMasterDetailEffect")]
namespace MasterDetail.iOS
{
    public class CustomMasterDetailEffect : PlatformEffect
    {
        protected override void OnAttached()
        {
            var page = Element as MasterDetailPage;
            page.Appearing += Page_Appearing;
        }

        protected override void OnDetached()
        {
            var page = Element as MasterDetailPage;
            page.Appearing -= Page_Appearing;
        }

        void Page_Appearing(object sender, EventArgs e)
        {
            var vc = GetParentViewController();
            var page = Element as MasterDetailPage;

            var navigationItem = vc.NavigationController.TopViewController.NavigationItem;
            navigationItem.LeftBarButtonItems = new UIBarButtonItem[]
            {
                new UIBarButtonItem("MENU", UIBarButtonItemStyle.Plain, (_, __) => 
                { 
                    page.IsPresented = !page.IsPresented; 
                })
            };
        }

        UIViewController GetParentViewController()
        {
            UIResponder responder = this.Container;
            while ((responder = responder.NextResponder) != null)
            {
                if (responder is UIViewController)
                {
                    return (UIViewController)responder;
                }
            }
            return null;
        }
    }
}
```

Effect は、 ``ResolutionGroupName`` と ``ExportEffect`` で定義した名称を使って、PCL側プロジェクトで Page に追加します。

```csharp RootPage.cs
public class RootPage : MasterDetailPage
{
    public RootPage ()
    {
        NavigationPage.SetHasBackButton(this, false);
        // Effect を追加する
        Effects.Add(Effect.Resolve("mycompany.CustomMasterDetailEffect"));

        // 以下省略

```

![](https://dl.dropboxusercontent.com/u/264530/qiita/customizing_left_navvigationbar_button_in_masterdetail_with_navigation_pages_05.png)

こちらも、CustomRenderer と同じことができました。

が、ちょっと黒魔術っぽいの使ってます。

* ``Effects.Container`` から取得できるのは ``UIView`` です。親の ``UIViewController`` を得るには、 ``GetParentViewController()`` でやってるような事をしなければなりません
* CustomRenderer はそれ自体は ``ViewController`` だったので ``ViewWillAppear()`` など画面のライフサイクルコールバックを override することができました。が、Effects から ViewController のライフサイクルイベントをハンドリングできません。代わりに Xamarin.Forms 側の ``Page`` のライフサイクルから ``Appearing`` イベントで処理するようにしています。そのため、「MENU」ボタンが表示されるタイミングが若干遅れます。

## CustomRenderer と Effects 、どちらを使えばいいの？

* CustomRenderer はできる事は多いが、複数の CustomRenderer を適用することはできない
* Effects はできる事は少ないが、複数の機能拡張を同時に適用できる

以上を考えると、

1. まずあなたの行いたいことが Effects で実現できないか、試してみる
2. Effects でできないレベルなら CustomRenderer を選択する

となるでしょう。

本件のネタは、 Effects ではかなりムリをして実現しているので、CustomRenderer の方が相応しいと思われます。
が、CustomRenderer はここぞという時にとっておきたい気もします。
このさじ加減は、作るアプリの規模・深度、汎用性、再利用性などによって変わってくるでしょう。Effects の方が汎用性・再利用性は高いですが、ネイティブのUIパーツをごっそり入れ替えるような深い事は、CustomRenderer でなければできません。

今回のプログラムは Github に上げてあります。（``CustomMasterDetailRenderer.cs`` はコメントアウトしてあって、Effects の方を活かしてます。）

* [XFNavigationWithMasterDetailSample/README.md at master · amay077/XFNavigationWithMasterDetailSample](https://github.com/amay077/XFNavigationWithMasterDetailSample/blob/master/README.md)

私は「Effects で頑張りたい派」かな。
