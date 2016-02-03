---
layout: post
title: "クロスプラットフォーム対応KVS Akavache を使ってお手軽にデータを保存する"
date: 2015-12-01 23:59:59 +0900
comments: true
categories: [Xamarin, Android, iOS]
---
　これは [Xamarin Advent Calendar 2015 1日目](http://qiita.com/advent-calendar/2015/xamarin) の記事です。

Xamarin Advent Calendar 2015、今日から開始です。
3年目になってもネタに尽きない Xamarin 、まだまだ盛り上がっております。

かくいう自分は、[前回の投稿](http://qiita.com/amay077/items/0a3fa3dfac7f29a2807d) が約1年前と、完全に時代遅れになっております（仕事とスプラ...いえ何でもないです）。
最近のトピックスは他の方にお任せして、1年前からのネタを書きます。
<!--more-->

Xamarin でギョームアプリを開発している時に、Android/iOS で使える ORM を探していたというか、JSON のデータをお手軽に保存・読み出し→インスタンス化できるライブラリないかなーと探していました。

Xamarin の公式ドキュメント

* [Cross-Platform Data Access - Xamarin](https://developer.xamarin.com/guides/cross-platform/application_fundamentals/data/)

には、 SQLite.NET や ADO.NET などが紹介されていますが、どれも面倒そう。そこで使ってみようと思ったのが [2年前の投稿](http://qiita.com/amay077/items/f14e04d4e86c8a782c15) でチラッと触れていた Akavache です。

# Akavache とは

* akavache/Akavache - https://github.com/akavache/Akavache

以下、README から引用です。

> Akavache is an asynchronous, persistent (i.e. writes to disk) key-value store created for writing desktop and mobile applications in C#, based on SQLite3. Akavache is great for both storing important data (i.e. user settings) as well as cached local data that expires.

(意訳)Akavacheは、C# による、SQLite3 をベースとした非同期で永続的なデスクトップとモバイルアプリケーション向けのキーバリューストアです。 ユーザー設定やキャッシュなどのローカルデータを保存するのに最適です。

バックエンドは SQLite3 なので、まあ得体の知れないデータ形式ではない、と。ちなみに [AkavacheExplorer](https://github.com/paulcbetts/AkavacheExplorer) というデータビューアもあります。

# 対応プラットフォーム

> Akavache is currently compatible with:
> 
> * Xamarin.iOS / Xamarin.Mac 32-bit
> * Xamarin.Android
> * .NET 4.5 Desktop (WPF)
> * Windows Phone 8
> * WinRT (Windows Store)
> * Windows Phone 8.1 Universal Apps

ほぼ全てやないかい！

# 使ってみよう

せっかくなので Xamarin.Form で Akavache を使ったアプリを作ってみます。

### 1. プロジェクト（ソリューション）の作成

Xamarin.Forms App で、新しいプロジェクトを作成します。プロジェクト名は AkavacheSample とでもします。

Shared Code: は、'Use Portable Class Library' を選択します。

プロジェクトが3つ（AkavacheSample, AkavacheSample.Droid, AkavacheSample.iOS）作成されます。

### 2. Akavache と、依存ライブラリの導入

**3つのプロジェクトそれぞれで** 、NuGet(メニュー → プロジェクト → Add NuGet Packages...) から、以下のパッケージを追加します（執筆時点の Akavache の最新バージョンは 4.1.2 です。）。

* Akavache.Core
* SQLitePCL.raw
* Akavache.SQLite3
* Akavache

検索ボックスに 「sqlite akavache」と入力すると全部表示されると思います(↓こんな感じに)。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_akavache_01.png)

追加に失敗する場合は、上のリストの順番で一つずつ追加するとうまくいくと思います。

### 3. サンプルアプリの画面を作る

サンプルアプリの画面レイアウトを作ります。XAML とか面倒なのでコードでバリッと。

AkavacheSample プロジェクトの App.cs を以下のようにします。テキストボックス２つとボタン２つが縦に並んでいるだけの簡単な画面です。


```csharp AkavacheSample.cs
public class App : Application
{
    public App()
    {
        var nameEntry  = new Entry { Placeholder = "名前を入力" };
        var ageEntry   = new Entry { Placeholder = "年齢を入力(数値のみ)" };
        var saveButton = new Button { Text = "保存" };
        var loadButton = new Button { Text = "読み出し" };
            
        // The root page of your application
        MainPage = new ContentPage
        {
            Padding = new Thickness(20),
            Content = new StackLayout
            {
                VerticalOptions = LayoutOptions.Center,
                Children =
                {
                    nameEntry,
                    ageEntry,
                    saveButton,
                    loadButton
                }
            }
        };
    }

    // 以下省略
}
```


### 4. Akavache を使って保存と読み出し

名前と年齢をひとまとめに保存したいので、Person というクラスを作ります。

```csharp Person.cs
public class Person
{
    public string PersonName { get; set; }
    public int PersonAge { get; set; }
}
```

あとはもう、一気に実装するだけです。
保存ボタンを押した時に、入力値を Person に詰めて、Akavache を使って保存します。
読み出しボタンを押した時に、Akavache から Person を読みだし、各テキストボックスにバラして設定します。

Akavache はキー・バリュー・ストアなので、保存・読み出し時のキーを ```"person"``` としています。


```csharp AkavacheSample.cs

public class App : Application
{
    public App()
    {
        var nameEntry  = new Entry { Placeholder = "名前を入力" };
        var ageEntry   = new Entry { Placeholder = "年齢を入力(数値のみ)" };
        var saveButton = new Button { Text = "保存" };
        var loadButton = new Button { Text = "読み出し" };

        saveButton.Clicked += async (sender, e) => 
        {
            // Person に詰めて…
            var person = new Person { 
                PersonName = nameEntry.Text, 
                PersonAge  = Convert.ToInt16(ageEntry.Text) 
            };

            // 保存
            await BlobCache.LocalMachine.InsertObject("person", person); 
        };

        loadButton.Clicked += async (sender, e) => 
        {
            // Akavache で Person を読み出し
            var loaded = await BlobCache.LocalMachine.GetObject<Person>("person");
            // 各テキストボックスに設定
            nameEntry.Text = loaded.PersonName;
            ageEntry.Text  = loaded.PersonAge.ToString();
        };
            
        // The root page of your application
        MainPage = new ContentPage
        {
            Padding = new Thickness(20),
            Content = new StackLayout
            {
                VerticalOptions = LayoutOptions.Center,
                Children =
                {
                    nameEntry,
                    ageEntry,
                    saveButton,
                    loadButton
                }
            }
        };
    }

    // 以下省略
}
```

### 5. 動かす！

Android Player と iOS Simulater で動かしてみた、の図です。(途中、iPhone でキーボード出すのに苦労してるところは無視してください、Take2 の時間が無かったのですｗ)

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_akavache_02.gif)

さすが Xamarin.Forms だ、(Android でも iPhone で動かしても)何とも無いぜ！

# まとめ

Akavache を使って保存と読み出しを行う超簡単なサンプルを作ってみました。
作ったサンプルは [GitHub - amay077/AkavacheSample](https://github.com/amay077/AkavacheSample) においておきます。

仕組みは理解できないけど、とても簡単に使えることが分かると思います。
追加されたパッケージを見ると、 JSON.NET とか、Reactive Extensions とかが入っているので、まあだいたい察しが付くかと。。。

ギョームアプリでがっつり使ってますが、今のところパフォーマンスとかデータ破損とか、そういう問題はないです。POCO なオブジェクトをローカル保存するのに、とても役立っています。

というわけで Advent Calendar 初日のネタは以上です。

