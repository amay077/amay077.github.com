---
layout: post
title: "Xamarin.Android で Reactive Extensions を使う"
date: 2013-03-01 17:38
comments: true
categories: [Xamarin, Android, C#, ReactiveExtensions]
---
なんか一気にハードル上がったような。。。
<!-- more -->

Reactive Extensions(Rx) については、

* [Reactive Extensionsの概要と利用方法 － ＠IT](http://www.atmarkit.co.jp/fdotnet/introrx/introrx_01/introrx_01_01.html)

などを読んで頂くとして、Rx は昨年 [オープンソース化され](http://www.infoq.com/jp/news/2012/11/rx-net-open-source)、本家を Mono for Android で使えるようにした

* [RxM4A - Mono For Android Reactive Extensions](http://rxm4a.codeplex.com/)

というプロジェクトがあります。
Mono で使えるということは Xamarin でももちろん…ということで使ってみます。

## サンプルアプリケーションを作る
とりあえず Xamarin.Android で "XamarinAndroidRxSample" というプロジェクトを作りました。

## RxM4A のソースを Clone する
上記のサイトからソースを取得します。以下のコマンドでもおｋ(要git)

> $git clone https://git01.codeplex.com/rxm4a

## ソリューションにプロジェクトを追加する

ソリューションツリーのソリューションのところで右クリック→追加→既存のプロジェクトを追加 

!["add_project"](https://dl.dropbox.com/u/264530/qiita/xamarin_android_rxm4a.png)

で、先ほど取得した RxM4A のディレクトリへ移動します。

!["projects"](https://dl.dropbox.com/u/264530/qiita/xamarin_android_rxm4a_projects.png)

RxMonoForAndroid の中から以下のプロジェクトを追加します。

* System.Reactive.Interfaces
* System.Reactive.Core
* System.Reactive.Linq

フォルダを掘ってくと xxx.csproj ってファイルがあるので、それを選択します。
依存関係があるので、上記の順番通りやらないとなにかエラーが出ますが、最終的に３つ揃えば問題ないでしょう。

## 参照を追加する
XamarinAndroidRxSample から、今追加した３つのプロジェクトを参照に追加します。

ソリューションツリーの XamarinAndroidRxSample のところで右クリック→参照アセンブリの追加 から、

!["add_project_reference"](https://dl.dropbox.com/u/264530/qiita/xamarin_android_rxm4a_add_reference.png)
!["add_project_reference"](https://dl.dropbox.com/u/264530/qiita/xamarin_android_rxm4a_add_reference2.png)

ダイアログのタブで Projects を選ぶと３つ出てくるのでチェックをいれて OK してください。

## Reactive Extensions を使ってみる

ボタンを押した時の処理で Rx を使ってみます。

```c# Main.cs
(ここまで省略)
using Android.Util;

using System.Reactive.Linq; // これ必要

namespace XamarinAndroidRxSample
{
    [Activity (Label = "XamarinAndroidRxSample", MainLauncher = true)]
    public class Activity1 : Activity
    {
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
                Observable.Range(0, 10) // 0〜9 のリスト
                .Where(n => n % 2 == 0) // 偶数だけ抽出
                .Select(n => n * 2) // 値を２倍して
                .Subscribe(n => Log.Debug("Activity1", n.ToString())); // 出力
            };
        }
    }
}
```

動かしてみた結果

!["output"](https://dl.dropbox.com/u/264530/qiita/xamarin_android_rxm4a_output.png)

全然 Rx っぽくない(Linq だけでできる) 処理ですけど、Rx 関連クラスが使えることは確認できました。

使い倒すには、RxMonoForAndroid 配下の他のプロジェクトを参照に追加して〜となるでしょう。

## アセンブリ参照はどうやるの？
ここではプロジェクト参照で行ったのですけど、各Rxプロジェクトをビルドしてできた

* System.Reactive.Interfaces.dll
* System.Reactive.Core.dll
* System.Reactive.Linq.dll

をアセンブリ参照してもいいでしょ？と思ってやってみました。
それぞれの System.Reactive.Linq プロジェクトの ``/bin/Debug`` ディレクトリの中に上記ファイルができているので、てっとり早くそれを XamarinAndroidRxSample でアセンブリ参照してみました。(ほんとはちゃんと Release ビルドしてね)

そしたらアプリは動くには動いたのですが、ソリューションツリーに怪しげなエラーが↓

!["error"](https://dl.dropbox.com/u/264530/qiita/xamarin_android_rxm4a_error.png)

アプリとDLLでターゲットフレームワークが違う(アプリは ICS4.0.3 でDLLは 4.0.0) だとこうなるんでしょうか？
Yes、アプリ側を 4.0.0 にしてプロジェクトクリーン→Xamarin Studio を再起動したら消えました。(けどアプリの方がバージョン上位だったのに、下位互換性はどうなってるんでしょ？)

## サンプルプロジェクト
これも github においておこう。RxM4A は Submodule として登録してみた。

* [amay077 / XamarinAndroid_RxSample](https://github.com/amay077/XamarinAndroid_RxSample) 

## 参考
* [Reactive Extensionsの概要と利用方法 － ＠IT](http://www.atmarkit.co.jp/fdotnet/introrx/introrx_01/introrx_01_01.html)
* [RxM4A - Mono For Android Reactive Extensions](http://rxm4a.codeplex.com/)
* [neue cc](http://neue.cc/) スーパー勉強になります
* [かずきのBlog@Hatena](http://d.hatena.ne.jp/okazuki/20111114/1321277465) 膨大なサンプル集がようやく Android開発で活かせます！
* [reactive4java](https://code.google.com/p/reactive4java/) Java ではこれ使ってました
