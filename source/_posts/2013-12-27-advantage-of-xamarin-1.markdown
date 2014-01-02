---
layout: post
title: "細かすぎて伝わらない Xamarin にして良かった事(その1)"
date: 2013-12-06 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#]
---
それ Xamarin じゃなくて C# じゃん！とかのツッコミはナシでｗ
まあ Java よりも Objective-C よりも C# がイイから選んでいるわけで。
サンプルコードは Xamarin.Android ですが iOS でも同じです。
<!--more-->
## 1. イベントのマルチキャストができる！

``setOnClickListener`` とかリスナー系が全部 ``Event`` になっているので、複数のリスナを登録可能、削除もできます。

```csharp MainActivity.cs
protected override void OnCreate(Bundle bundle)
{
    /** 省略 **/

    Button button = FindViewById<Button>(Resource.Id.myButton);

	// Label 変えます
    button.Click += delegate
    {
		button.Text = string.Format("{0} clicks!", count++);
    };

	// Logcat にも出しちゃう
	button.Click += (sender, e) => Android.Util.Log.Debug("Main", "Clicked!");

	// .NET2.0 な方の書き方
	button.Click += button_Click;
}

// Toast にも出そ
void button_Click(object sender, EventArgs e)
{
	Toast.MakeText(this, "Clicked!", ToastLength.Short).Show();
}
```

## 2. ``var`` が使える！

C# なら ``var`` 使わなきゃ。

```csharp using_var
Dictionary<String, Object> map = new Dictionary<Piyo, Hoge>(); // 長いよ…
var map = new Dictionary<Piyo, Hoge>();

//Button button = FindViewById<Button>(Resource.Id.myButton);
var button = FindViewById<Button>(Resource.Id.myButton);
```

Variant じゃないですから、念の為。

## 3. Runnable がラムダ式で書ける！

Android-Java では ``Activity.runOnUiThread`` や ``Handler.post`` って Runnable を受け取るようになっていて、大抵無名クラスにするので、長ったらしい記述になってしまいますが、Xamarin.Android では、Runnable に加えて ``Action`` も受け取ってくれ、これはラムダ式で書けるので非常にスッキリ書けます。
Runnable にかぎらずラムダ式の恩恵は大きいのですが(イベントハンドラとか)。

```csharp UsingLambdaInsteadOfRunnable.cs
var button = FindViewById<Button>(Resource.Id.myButton);

button.Click += (s, e) =>
{
    button.Text = "Progress...";

    Task.Factory.StartNew(() =>
    {
        // Fat な処理
        Thread.Sleep(3000);

        // UIスレッドで実行
        this.RunOnUiThread(() => button.Text = "Finished.");
        /*// Java だとこんな長ったらしいコードが書かないといけない
        MainActivity.this.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // hogehoge
            }
        }); */
    });
};
```

ちょっと話がそれますが、``Activity.runOnUiThread`` 自体、Android 固有の API なので、プラットフォーム依存を減らそうと思ったら ``SynchronizationContext`` を使います。

```csharp UsingSynchronizationContext.cs
var button = FindViewById<Button>(Resource.Id.myButton);

var syncContext = SynchronizationContext.Current;
button.Click += (s, e) =>
{
    button.Text = "Progress...";

    Task.Factory.StartNew(() =>
    {
        // Fat な処理を別スレッドで
        Thread.Sleep(3000);
    }).ContinueWith(t => syncContext.Post(state =>
    {
        // UIスレッドでの処理
        button.Text = "Finished.";
    }, null));
};
```

## 4. async/await が使える！

それはそうと上のコード、async/await を使ったらたったの3行ですよ！
Android の ``AsyncTask`` とか、iOS の GCD とか、ほぼ捨てられますよ。

```csharp UsingAsyncAwait.cs
var button = FindViewById<Button>(Resource.Id.myButton);

button.Click += async (sender, e) => 
{
    button.Text = "Progress..."; // UIスレッド
    await Task.Run(() => Thread.Sleep(3000)); // ワーカースレッド
    button.Text = "Finished."; // UIスレッド
};
```

とりあえずこんなところで。
他にも、気づいたら書きます。