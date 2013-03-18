---
layout: post
title: "Xamarin の Alpha版で async/await を試す"
date: 2013-03-18 20:42
comments: true
categories: [Xamarin, C#, Android]
---
Xamarin Blog で "[Alpha版だけど async/await 使えるようになったよー](http://blog.xamarin.com/brave-new-async-mobile-world/)" との事だったのでさっそく試してみました。
<!--more-->
以前から、["Good news – we plan to release full support for async/await in all our products in April of 2013."](http://xamarin.uservoice.com/forums/144858-xamarin-suggestions/suggestions/2697497-async-await-support) と言われていたので予定どおりですかね。

## Alpha版をインストール
Xamarin Studio のシステムメニュー → アップデートをチェック で、 "Update channel" で **Alpha** を選ぶとインストールできます。もちろん、アルファ版なので自己責任で。

インストール後の、Xamarin と各SDK のバージョンはこう↓なってました。

>Xamarin Studio
>Version 4.0.2 (build 18)
>Installation UUID: xxxxxx
>Runtime:
>	Mono 3.0.7 (master/514fcd7)
>	GTK 2.24.16
>	GTK# (2.12.0.0)
>	Package version: 300070000
>
>Apple Developer Tools
>Xcode 4.6 (2066)
>Build 4H127
>
>Xamarin.Mac
>Xamarin.Mac: Not Installed
>
>Xamarin.Android
>Version: 4.7.0 (Trial Edition)
>Android SDK: /Users/hrnv/dev/sdks/android-sdk-macosx
>
>Xamarin.iOS
>Version: 6.3.0.255 (Trial Edition)
>Hash: ba05545

## 同期処理だと…

「ボタンを押すと **超時間のかかる処理** を実行して、結果を表示する」というケースで試してみます。

まず何も考えず同期処理で書くと、、、

```c# sync.cs
// ボタンが押されたよ
private void button1_Click(Object sender, EventArgs e)
{
    button.Enabled = false; // 実行中はボタン使えなくする
    
    // 超時間のかかる計算
    var result = HeavyCalc();
    
    button.Text = String.Format("result:{0}", result); // 結果を表示する
    button.Enabled = true;
}

// 超時間のかかる計算
private static int HeavyCalc()
{
    System.Threading.Thread.Sleep(10000);
    return 5; // 超時間をかけて 5 を計算したつもり
}
```

こんな感じ。
動かしてみます。

!["anr"](https://dl.dropbox.com/u/264530/qiita/xamarin_async_await_preview_anr.png)

あえなくフリーズ＆ANR、当然です。

## async/await 化

async/await については、ググればたくさん情報が出てきますが、探した中でもっとも簡単とおもわれる例を紹介します。

* [[C#]async/awaitの使い方メモ、その１。 | Kimux.Net](http://kimux.net/?p=902)

さて、先ほどのプログラムを、async/await 構文を使って非同期化してみます。

```c# async.cs
// ボタンが押されたよ
private async void button1_Click(Object sender, EventArgs e)
{
    button.Enabled = false; // 実行中はボタン使えなくする
    
    // 超時間のかかる計算
    var result = await HeavyCalcAsync();
    
    // ここから下は UIスレッド で実行される
    button.Text = String.Format("result:{0}", result); // 結果を表示する
    button.Enabled = true;
}

// 超時間のかかる計算
private static int HeavyCalc()
{
    System.Threading.Thread.Sleep(10000);
    return 5; // 超時間をかけて 5 を計算したつもり
}

// HeavyCalc をラップして非同期で実行
private Task<int> HeavyCalcAsync()
{
    return Task.Run(() => HeavyCalc());
}
```

変更点を、ソースの下の方から。
まず、``HeavyCalc`` をラップして、 ``HeavyCalcAsync`` という関数を作りました。``Task`` クラスを使って非同期で ``HeavyCalc`` を実行する処理です。async/await のルールに従って ``Task`` クラスを返値にします。メソッド名のおしりに "Async" を付けるのもルールです。

次に、 ``button1_Click`` です。メソッドの定義に ``async`` キーワードを付けます。このメソッドが非同期である事を示すと共に、メソッド内に ``await`` キーワードが含まれる事を意味します。メソッド内に ``await`` が無いとエラーになります。(Xamarin Studio でもちゃんとエラーにしてくれました)

最後に、「超時間のかかる計算」の呼び出し。``HeavyCalc`` の代わりに ``HeavyCalcAsync`` を ``await`` 付きで記述します。

これで終わり。
「button1_click は、非同期で HeavyCalcAsync を実行し、その終了を待って、その後続処理を **UIスレッド** で続行する」という意味になりました。

動かしてみます。

!["async"](https://dl.dropbox.com/u/264530/qiita/xamarin_async_await_preview_asyc.png)

ANR でませんし、ちゃんと計算後に画面が更新されます。

## AsyncTask で書くと…

Android で非同期処理と言えば ``AsyncTask`` がよく紹介されてますので、一応、それを使うとどうなるのか書いてみます。

まず、AsyncTask を拡張して、HeavyCalc をバックグラウンドで実行する ``HeavyCalcTask`` を用意します。
``OnPreExecute`` と ``OnPostExecute`` でボタンを無効/有効 にしています。

```c# HeavyCalcTask.cs
// HeavyCalc を非同期で実行する AsyncTask
class HeavyCalcTask : Android.OS.AsyncTask
{
    private readonly Button button;

    public HeavyCalcTask(Button button)
    {
        this.button = button;
    }

    #region implemented abstract members of AsyncTask
    protected override void OnPreExecute()
    {
        button.Enabled = false; // 実行中はボタン使えなくする
    }

    protected override Java.Lang.Object DoInBackground(params Java.Lang.Object[] @params)
    {
        // 超時間のかかる計算
        return HeavyCalc();
    }

    protected override void OnPostExecute(Java.Lang.Object result)
    {
        button.Text = String.Format("result:{0}", result); // 結果を表示する
        button.Enabled = true;
    }
    #endregion
}
```

使う方は、まあ普通に。

```c# asynctask_execute.cs
// ボタンが押されたよ
private async void button1_Click(Object sender, EventArgs e)
{
    var asyncTask = new HeavyCalcTask(button);
    asyncTask.Execute();
}
```

クラスを作らなきゃいけないし、AsyncTask の仕方ないところですけど、View に対する処理とロジックが同じクラスに同居しちゃうし、複数の非同期処理を逐次実行できないし、とあまり良い所が見えません。

そもそも Xamarin.iOS などとクロスプラットフォームを考えるならプラットフォーム固有の機能の利用は最小限に留めたいので、Xamarin.Android で AsyncTask を使う意味は「ない」でしょう。
(Xamarin.iOS でも async/await は使えるそうです。)

## まとめ
* 知らない人がコード見ると「これ await してたら ANR 起こりますよね？」とか「これ HeavyCalcAsync が非同期だから、次の行がすぐ実行されちゃいますよね？」とか言われそうｗ
* 諸君らが愛してくれた ``AsyncTask`` は死んだ
* async/await が使えるようになるまでは ``Task.Factory.StartNew`` で。

## 参考
* [Xamarin previews C# async on iOS and Android | Xamarin Blog](http://blog.xamarin.com/brave-new-async-mobile-world/)
* [[C#]async/awaitの使い方メモ、その１。 | Kimux.Net](http://kimux.net/?p=902)
* [async/await : xin9le note](http://xin9le.net/archives/tag/async-await/page/2)
* [.NET開発における非同期処理の基礎と歴史 － ＠IT](http://www.atmarkit.co.jp/fdotnet/chushin/masterasync_01/masterasync_01_01.html)