---
layout: post
title: "Xamarin.Forms の Label から iOS の UILabel を取り出す"
date: 2014-06-13 15:34
comments: true
categories: [Xamarin, iOS, C#]
---
[Xamarin.Forms でどうにかしたい iOS と Android の違い]( http://qiita.com/amay077/items/12979585ac3e2dcacacb) の「文字の自動縮小」の自己回答。
<!--more-->

Xamarin.Forms で定義した ``Label`` は、iOS では ``UILabel`` となるはずなので、その過程のどこかでフックできれば ``UILabel.AdjustsFontSizeToFitWidth`` が仕込める、と目論んで、ホントにできたのでメモ。

## 要点

~~Forms→ネイティブのフックは PageRenderer でできる。その中で得られる UIView（のサブクラス）は、Label と UILabel の両方の参照を持っているので、あとは使うだけ。~~

**ページでなく、UIパーツレベルでフックできたので、全面的に書き換えた。**

## やってみる

参考にしたのは https://github.com/xamarin/xamarin-forms-samples/tree/master/Forms2Native 。

このサンプルをちょっと改造して試した。

まずは Forms側の MySecondPage.cs を修正。

```csharp MySecondPage.cs
public class MySecondPage : ContentPage
{
    public Label MyLabel { get; private set; }

	public MySecondPage ()
	{
        this.MyLabel = new Label
        {
            Text = "Too loooooooooooooooooooooooong label",
            Font = Font.SystemFontOfSize(30d),
            LineBreakMode = LineBreakMode.NoWrap
        };

        Content = new StackLayout
        {
            Orientation = StackOrientation.Vertical,
            VerticalOptions = LayoutOptions.CenterAndExpand,
            Children = 
            {
                this.MyLabel
            }
        };
	}
}
```

ラベルを配置。とても文字が長いので全部は表示しきれない。

次に iOS側に MyLabelRenderer.cs を作成。

```csharp MyLabelRenderer.cs
using System;
using Xamarin.Forms;
using Forms2Native;
using Xamarin.Forms.Platform.iOS;
using MonoTouch.UIKit;

[assembly:ExportRenderer(typeof(Label), typeof(MyLabelRenderer))]

namespace Forms2Native
{
    public class MyLabelRenderer : LabelRenderer
    {
        protected override void OnElementChanged(ElementChangedEventArgs<Label> e)
        {
            base.OnElementChanged(e);
            this.Control.AdjustsFontSizeToFitWidth = true;
        }
    }
}
```

``ExportRenderer`` で「Formsの``Label``は、``MyLabelRenderer``を使う」と定義している。
するとすべての ``Label`` の生成時を ``OnElementChanged`` でフックでき、``Control`` で ``UILabel`` は取り出せるので、あとはご自由に、という感じ。

この実装だと、すべての Label に Ajusts が適用されてしまう。個別に行いたい場合は、Forms側に Labelから派生した ``AjustableLabel`` を作成して使い、``ExportRenderer(typeof(Label),…`` のところを ``ExportRenderer(typeof(AjustableLabel),…`` にすればいけるはず。そしてこの方法はカスタムビューを作る手順に通じる（というかそのもの？）はず。

ちなみにこの ``OnElementChanged`` は、Nuget の Xamarin.Formsパッケージの Ver1.1.0.6201から利用できる。

## 実行する

こんな感じで、ちゃんと文字サイズが縮小されました。

![](https://dl.dropboxusercontent.com/u/264530/qiita/getting_uilabel_from_xamarin_forms.png)

Android の方も同じ要領でいけるは…ず。
