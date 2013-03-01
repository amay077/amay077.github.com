---
layout: post
title: "Xamarin.Android で作った HelloWorld のソースを眺めてみる"
date: 2013-02-28 00:38
comments: true
categories: [Android, Xamarin, C#]
---
MonoDroid とか Mono for Android とか呼ばれてた時は、「あーどうせ MonoDevelop と他のモジュールあれこれインストールしなきゃいけないんでしょ？」と腰が重かったのですが、[Xamarin 2.0 としてオールインワン化](http://www.forest.impress.co.jp/docs/news/20130221_588816.html)されるとこうも食指が動きますか。

<!-- more -->

さっそく Xamarin.Android のプロジェクトを作って、中身をみてみました。

## ダウンロード
[Xamarin.com](http://xamarin.com/) から **Download Now** しましたよ。あ、環境は Mac(Lion) です。

## インストール
dmg 開いてインストーラっぽいのを実行するだけ。Android と iOS の SDK の場所を聞かれましたがデフォのまま続行しました。たぶん Android 開発者はもう SDK あるのでそこを指定しても良いのでしょう。

## 起動、プロジェクト作成

C# - Android - Android Ice Cream Sandwitch Application を選択。

!["create_project"](https://dl.dropbox.com/u/264530/qiita/xamarin_create_project.png)

ちなみに VB.NET もありますが、Android用のプロジェクトテンプレがありませんでした。

## プロジェクトの中身はこんな感じ

!["project"](https://dl.dropbox.com/u/264530/qiita/xamarin_helloworld.png)

ソリューションツリー、Visual Studio っぽさと Android っぽさが同居していてなんか不思議な感じです。

「参照」「Properties」「Resource.designer.cs」あたりは Visual Studio っぽいですね。
一方、「Assets」「drawable」「layout」「values」あたりは Android っぽいですね。
ちなみに、``layout/Main.axml`` というファイルがあります。一瞬「ザムル(Xaml)？」と見間違えましたが、開いてみるとなんのことはない、ただの Android Layout XML でした(^_^;)

そして ``MainActivity.cs`` 、混血ですね。

ソースを見てみましょう。

```c# MainActivity.cs
using System;

using Android.App;
using Android.Content;
using Android.Runtime;
using Android.Views;
using Android.Widget;
using Android.OS;

namespace HelloXamarinAndroid
{
    [Activity (Label = "HelloXamarinAndroid", MainLauncher = true)]
    public class Activity1 : Activity
    {
        int count = 1;

        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);

            // Set our view from the "main" layout resource
            SetContentView(Resource.Layout.Main);

            // Get our button from the layout resource,
            // and attach an event to it
            Button button = FindViewById<Button>(Resource.Id.myButton);
			
            button.Click += delegate
            {
                button.Text = string.Format("{0} clicks!", count++);
            };
        }
    }
}
```

ソースの中にも Android の要素がたくさん確認できます。

``[Activity (Label = "HelloXamarinAndroid", MainLauncher = true)]``
これ、本家では AndroidManifest.xml の Intent-Filter に定義する設定ですね。ここに書けちゃうみたいです。

``public class Activity1 : Activity``
Activity クラスを継承するのも変わりません。

``protected override void OnCreate(Bundle bundle)``
onCreate を override するのも、その中で setContentView するのも、findViewById するのも本家と変わりません。C# っぽくメソッド名が大文字で始まっているのと、若干メンバ名(R.id が Resource.Id とか)が変わっているくらいです。

そしてボタンクリックらへんの処理、

```c#
Button button = FindViewById<Button>(Resource.Id.myButton);
			
button.Click += delegate
{
    button.Text = string.Format("{0} clicks!", count++);
};
```

C# ならではの匿名delegate 使ってます。OnClickListener インターフェースを実装しなければならない本家に比べて短く書けます。
更に短くするなら以下でしょうか。

```c#
var button = FindViewById<Button>(Resource.Id.myButton);
            
button.Click += (sender, e) => 
    button.Text = string.Format("{0} clicks!", count++);
```

まず ``var``。型推論ですよ Variant じゃないですよ。
そしてイベントハンドラはラムダ式で１行で書けます。いや嬉しい。

あと地味にイベントハンドラが += で複数追加できるのもありがたいと思う時が来るでしょう。

## ビルド、実行
実行すると、こんなダイアログが出てきます。

!["devices"](https://dl.dropbox.com/u/264530/qiita/xamarin_device_select.png)

実機もちゃんと認識されます。

## まとめ
ソースコード見るまで勘違いしてました。クロスプラットフォームをうたっているから HelloWorld くらいなら "Write once, run anywhere" なのかと。

全然違いました。Xamarin.Android は気持ち良いくらいに Android SDK のラッパですし、Xamarin.iOS は iOS SDK のラッパでした。

UI は共通化できませんし、プラットフォーム固有の機能を使うロジックも共通化できません。M-V-VM なら、共通化できるのは M の一部と VM くらい？それも ``DependencyProperty`` みたいなのは用意されていないので自作する必要があります。

しかしラッパなだけに元々 Android の開発をしていた人にとっての学習コストは低いです。
Obj-C やりたくねー、って思ってた人にもちょうど良いかも知れません。

なんといっても、スマホアプリ開発で async/await とか Reactive Extensions とか使えるのかと思うと wktk です(^^)

最後に Xamarin.android も Xamarin Studio も、ここまで完成度が高いとは思ってませんでした。Xamarin さん、Mono さんごめんなさい。(ソースコードエディタで日本語入力ができないのは僕だけでしょうか？)
なんか C# 楽しいので[買っちゃい](https://store.xamarin.com/)そうです、INDEE くらいなら。

## 参考

* [Xamarin 2.0 | ++C++; // 未確認飛行 C ブログ](http://ufcpp.wordpress.com/2013/02/24/xamarin-2-0/)

Mono ランタイムが同梱されているのですね。上の HelloWorld で 1.2MB くらいでした。