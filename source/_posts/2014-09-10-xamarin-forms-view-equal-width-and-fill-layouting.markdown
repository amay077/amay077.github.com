---
layout: post
title: "Xamarin.Forms で複数ボタンの均等割り付けをする"
date: 2014-09-10 22:57:08 +0900
comments: true
categories: [Xamarin, Android, iOS]
---
　Android では、例えば画面の幅に対して、複数のボタンを同じ幅でいい感じに Fill させることができます。(これを均等割り付けというのが正しいのかはよくわかりませんが)
<!--more-->
* [LinearLayout を使って均等割り付け : layout_weight - 戌印-INUJIRUSHI- （Androidあれこれ） -](http://inujirushi123.blog.fc2.com/blog-entry-106.html)
* [Androidレイアウトの要点だけ: LinearLayoutでパーツを均等に配置したい | スマートフォン要点だけブログ](http://blog.imho.jp/2011/08/android-linearlayout.html)

これで画面の解像度が違っても、横向きになっても、同じ幅のボタンで埋まる、という事ができます。


[Xamarin.Forms](http://xamarin.com/forms) でこれを実現するにはどうしたら良いか、試してみました。

## StackLayout を利用した試み

Xamarin.Forms では、LinearLayout に相当するレイアウトとして [StackLayout](http://iosapi.xamarin.com/?link=T%3aXamarin.Forms.StackLayout) があります。

まずはこれを利用してみます。

``StackLayout`` を ``Orientation = Horizontal`` とし、``Children`` にボタンを3つ配置しています。ボタンの幅は全て ``HorizontalOptions = FillAndExpand`` とします。

```csharp App.cs
public class App
{
    public static Page GetMainPage()
    {	
        return new ContentPage
        { 
            Content = new StackLayout
            {
                HorizontalOptions = LayoutOptions.FillAndExpand,
                VerticalOptions = LayoutOptions.Center,
                Orientation = StackOrientation.Horizontal,
                Children = 
                {
                    new Button
                    {
                        VerticalOptions = LayoutOptions.Center,
                        HorizontalOptions = LayoutOptions.FillAndExpand,
                        Text = "one",
                        TextColor = Color.Black,
                        BackgroundColor = Color.Aqua,
                    },
                    new Button
                    {
                        VerticalOptions = LayoutOptions.Center,
                        HorizontalOptions = LayoutOptions.FillAndExpand,
                        Text = "two two",
                        TextColor = Color.Black,
                        BackgroundColor = Color.Fuschia,
                    },   
                    new Button
                    {
                        VerticalOptions = LayoutOptions.Center,
                        HorizontalOptions = LayoutOptions.FillAndExpand,
                        Text = "three three three",
                        TextColor = Color.Black,
                        BackgroundColor = Color.Lime,
                    },
                },
            },
        };
    }
}
```

　これを iOS/Android 双方で実行すると、こうなりました。

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_forms_view_equal_width_and_fill_layouting_01.png)

うーん、そうじゃない（汗
どうやらボタンのテキストが全て表示されるように頑張ってくれちゃうようです。
まあ、これはこれで使える感じもします。

## Grid を利用した試み

他のレイアウトで考えてみます。Android の TableLayout に相当する [Grid](http://iosapi.xamarin.com/?link=T%3aXamarin.Forms.Grid) を使ってみます。

さっきの ``App.cs`` の実装を次のように変えます。

``Grid`` に1行3列の表を定義します。
列の定義 ``ColumnDefinition`` で幅を ``new GridLength(1, GridUnitType.Star)`` としているのは、「3列とも同じ比率の幅とする」ことを意味しています。(ということは比率を2:1:1にしたければ、最初の列だけ``2``にすればOKです)

```csharp App.cs
public class App
{
    public static Page GetMainPage()
    {	
        var grid = new Grid
        {
            HorizontalOptions = LayoutOptions.FillAndExpand,
            VerticalOptions = LayoutOptions.Center,
            RowDefinitions =
            {
                new RowDefinition() { Height = GridLength.Auto }
            },
            ColumnDefinitions = 
            {
                new ColumnDefinition() { Width = new GridLength(1, GridUnitType.Star) },
                new ColumnDefinition() { Width = new GridLength(1, GridUnitType.Star) },
                new ColumnDefinition() { Width = new GridLength(1, GridUnitType.Star) },
            }
        };

        grid.Children.Add(new Button
        {
            VerticalOptions = LayoutOptions.Center,
            HorizontalOptions = LayoutOptions.FillAndExpand,
            Text = "one",
            TextColor = Color.Black,
            BackgroundColor = Color.Aqua,
        }, 0, 0);

        grid.Children.Add(new Button
        {
            VerticalOptions = LayoutOptions.Center,
            HorizontalOptions = LayoutOptions.FillAndExpand,
            Text = "two two",
            TextColor = Color.Black,
            BackgroundColor = Color.Fuschia,
        }, 1, 0);

        grid.Children.Add(new Button
        {
            VerticalOptions = LayoutOptions.Center,
            HorizontalOptions = LayoutOptions.FillAndExpand,
            Text = "three three three",
            TextColor = Color.Black,
            BackgroundColor = Color.Lime,
        }, 2, 0);

        return new ContentPage
        { 
            Content = grid,
        };
    }
}
```

　これを iOS/Android 双方で実行すると、こうなりました。

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_forms_view_equal_width_and_fill_layouting_02.png)

　おーけー、意図したレイアウトになりました。ボタンに入りきらないテキストはiOSだと省略され、Androidだと折り返されるという違いはありますが、レイアウトの一貫性は保つことができました。

　ちなみに横向きにしても大丈夫です。

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_forms_view_equal_width_and_fill_layouting_03.png)

## まとめ

　Xamarin.Forms でも、Android+LinearLayoutのような均等配置ができました。
　クロスプラットフォームなので、iOSでも同じように動作します。
iPhone6 が発表されてiOS開発でも多解像度対応が必須になるので、これは有用な感じがします。
（というか Storyboard の AutoLayout では、これと同じことができる気がしないのですが。。。）


