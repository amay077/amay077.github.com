---
layout: post
title: "UIAlertController を async/await 対応させて便利に使う"
date: 2014-12-24 00:00:00 +0900
comments: true
categories: [Xamarin, iOS, C#]
---
　これは [Xamarin Advent Calendar 2014 23日目](http://qiita.com/advent-calendar/2014/xamarin) の記事です。

　なんか空いてたのでエントリーしましたが、急だったので軽い話です。
<!--more-->

* [iOS アプリでアラート出してボタンが押されるまで待つ？方法を Objective-C と Xamarin.iOS で比べてみた - Qiita](http://qiita.com/amay077/items/56abeeaa188f33cd56de)

の焼き直しみたいなものです。

　iOS8 では、``UIAlertDialog`` が非推奨になり、代わりに ``UIAlertController`` を使えとのこと。

　普通に使うとこうなります。

```csharp
button1.TouchUpInside += (sender, e) => 
{
    var alert = UIAlertController.Create("", "こんぼう をすてますか？", UIAlertControllerStyle.Alert);
    alert.AddAction(UIAlertAction.Create("はい", 
        UIAlertActionStyle.Default, x=> label1.Text = "こんぼう をすてました")); 
    alert.AddAction(UIAlertAction.Create("いいえ",  
        UIAlertActionStyle.Default, x=> {})); 
     
    this.PresentViewController(alert, true, null);
};
```

　このくらいなら問題ありません。

　次に、こんぼう をすてる前にもう一度問いかけるようにします。
２つ目の ``UIAlertController`` が入れ子になってしまって見づらい、 **残念な感じ** です。

```csharp
button1.TouchUpInside += (sender, e) => 
{
    var alert = UIAlertController.Create("", "こんぼう をすてますか？", UIAlertControllerStyle.Alert);
    alert.AddAction(UIAlertAction.Create("はい", 
        UIAlertActionStyle.Default, x=> 
        {
            // 念押しの確認ダイアログ（入れ子でつらい
            var alert2 = UIAlertController.Create("", "ほんとうにすてますか？", UIAlertControllerStyle.Alert);
            alert2.AddAction(UIAlertAction.Create("もちろん", UIAlertActionStyle.Default, _=> 
            {
                label1.Text = "こんぼう をすてました"
            }));
        alert2.AddAction(UIAlertAction.Create("やめる",  UIAlertActionStyle.Default, _=> {})); 

        // アラート２の表示
        this.PresentViewController(alert2, true, null);
    })); 

    // アラート１の表示
    alert.AddAction(UIAlertAction.Create("いいえ",  UIAlertActionStyle.Default, x=> {})); 

    this.PresentViewController(alert, true, null);
};
```

　Objective-C や Swift なら、ここで打つ手は今のところ無いでしょう。
しかし **Xamarin には、C# には async/await がありまぁす！** 
アラートの表示を async/await（というか Task）対応してみましょう。

```csharp
private Task<int> ShowDialog(string message, string button1Title, string button2Title)
{
    var comp = new TaskCompletionSource<int>();

    var alert = UIAlertController.Create("", message, UIAlertControllerStyle.Alert);
    alert.AddAction(UIAlertAction.Create(button1Title, UIAlertActionStyle.Default, x=> 
    {
        comp.SetResult(1); // OKボタン
    })); 
    alert.AddAction(UIAlertAction.Create(button2Title,  UIAlertActionStyle.Default, x=> 
    {
        comp.SetResult(0); // Cancel
    })); 

    this.PresentViewController(alert, true, null);

    return comp.Task;
}
```

``Task<int>`` を返すメソッド ``ShowDialog`` です。``UIAlertController`` のボタンが押されたら ``SetResult`` して Task の値を決定します。

　このメソッドを使う方は、こうなります。

```csharp
button1.TouchUpInside += async (sender, e) => 
{
    if (await ShowDialog("こんぼう をすてますか？", "はい", "いいえ") == 0) 
        return;

    if (await ShowDialog("ほんとうにすてますか？", "もちろん", "やめる") == 0) 
        return;

    label1.Text = "こんぼう をすてました";
};
```

なんて見やすいコードになったことでしょう。すばらしい！

入れ子でなく、フラットに書けるので、こんな事もできます。

```csharp
button1.TouchUpInside += async (sender, e) => 
{
    while (await ShowDialog("こんぼう をすてますか？", "はい", "いいえ") == 1) 
    {
        label1.Text = "それをすてるなんてとんでもない！";
    }

	label1.Text = "すてるのをやめました";
};
```

こんぼうを捨てるのをあきらめるまで、なんどでも聞いてきます。
コールバックスタイルのメソッドでループとか、ベタに書くと頭痛いです。

動かすとこんな感じです。

![](https://dl.dropboxusercontent.com/u/264530/qiita/uialertcontroller_with_async_await_01.gif)

``ShowDialog`` は拡張メソッドとして作成しておくと、呼び出しに便利かもしれません。
コールバックスタイルの機能を、Task化するパターンはよく使いそうな気がします。``TaskCompletionSource``、覚えておきましょう。

