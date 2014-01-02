---
layout: post
title: "Xamarin に Reactive Extensions を導入する"
date: 2013-12-12 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#, ReactiveExtensions]
---
昨日の [ポスト](http://qiita.com/amay077/items/49681b2de5d8cf208112) を使ったのに、肝心の導入部分を説明するのを忘れていました。まあ Components から追加するだけなのですが。他のアドベントカレンダーとの掛け持ちで疲れたので、今日は軽く書いて済ませます。
<!--more-->
## Reactive Extensions を導入する

Components で右クリック → Get more components → Reactive とかで検索 → 見つけたら Add to App で OK です。あ、この手順は .iOS でも .Android でも同じです。

![](https://dl.dropboxusercontent.com/u/264530/qiita/install_rx_to_xamarin_ios_01.gif)

## 使ってみましょうか

イベントを Stream に変換する例を示してお茶を濁します。

まずこんな感じのどうでもいい画面を用意しまして、

![](https://dl.dropboxusercontent.com/u/264530/qiita/install_rx_to_xamarin_ios_02.png)

``UIButton.TouchUpInside`` を ``IObservable`` に変換する拡張メソッドを用意します。

```csharp UIButtonExtensions.cs
public static class UIButtonExtensions
{
    public static IObservable<string> ClickAsObservable(this UIButton button)
    {
        return Observable.FromEventPattern<EventArgs>(button, "TouchUpInside")
                .Select(e => ((UIButton)e.Sender).TitleLabel.Text);
    }
}
```

で、こんなコードを書きます。

```csharp MainViewController_amb.cs
public override void ViewDidLoad()
{
    base.ViewDidLoad();

    Button1.ClickAsObservable()
        .Publish(_ => 
            Button2.ClickAsObservable()
            .Amb(
                Button3.ClickAsObservable()))
        .Subscribe(btnName => InvokeOnMainThread(() => 
            Label1.Text = btnName + " Clicked"));
}
```

### Observable.Amb (先にきた値を流す)

Button1 を押すと処理の開始です。
Publish で分配して **Amb** は Button2 のクリックと Button3 のクリックで先に行われた方を Label1 に表示します。

### Observable.Zip (どちらの値も待つ)

Amb を Zip に変えてみます。

```csharp MainViewController_zip.cs
    Button1.ClickAsObservable()
        .Publish(_ => 
            Button2.ClickAsObservable()
            .Zip(
                Button3.ClickAsObservable(), (l,r) => new {l, r}))
        .Subscribe(p => InvokeOnMainThread(() => 
            Label1.Text = p.l + " and " + p.r + "Clicked"));
```

Zip は2つのシーケンスの結果を待ってから後続へ流します。
なので、Button2 と Button3 の両方が押されると、Label1 に表示されます。

どちらもフツーに書くとフラグ変数なんか使って実現すると思うんですけど、Rx を使うと読みやすいコードになると思います。

あと、私は Android-Java の開発では [reactive4Java](https://code.google.com/p/reactive4java/) を使ってるんですが、あれには ``Observable.FromEventPattern`` が無い(Java のイベントの Listener はイベントのマルチキャストに対応してないから仕方ない)ので、Xamarin にすることで「完全な」Rx を使うことができて、こりゃ勉強のしがいがあるなあと思ったのでした。

何の記事でしたっけ？という感じですけど Xamarin なら Linq も Rx も使えてハッピーという事で、今日は以上です。