---
layout: post
title: "Pull up to Close を実装してみる、Xamarin.iOS で"
date: 2013-07-28 13:51
comments: true
categories: [Xamarin, iOS, C#]
---
イマドキのスマホアプリでは Pull to Refresh（引っ張って更新）を実装してるアプリをよく目にするのですが、RSS Reader の Feedly では Pull up to Close（上に引っぱって閉じる）も採用しています。

この操作性がなかなか使いやすかったので、自分でも実装してみました。
<!--more-->
## デモ

こんな感じ。
WebView なんですが、一番下までスクロールして、さらに上に引っ張ると "Pull up to Close" → "Release to Close" とラベルが変わり、そこで離すとコールバックします。

<iframe width="640" height="360" src="http://www.youtube.com/embed/AP6xPqwwXMI?feature=player_detailpage" frameborder="0" allowfullscreen></iframe>

## 実装してみたコード

[Xamarin.iOS](http://xamarin.com/) ですから、C# です。

UIWebView でやってますが、ScrollView なコントロールならだいたい同じ感じでいけるんじゃないかと思います。

```c# PullUpToCloseSampleViewController.cs
public partial class PullUpToCloseSampleViewController : UIViewController
{
    public PullUpToCloseSampleViewController() : base ("PullUpToCloseSampleViewController", null)
    {
    }

    public override void ViewDidLoad()
    {
        base.ViewDidLoad();

        // WebView が持ってる ScrollView、よく使うので変数化しておく
        // webView は Interface Builder で UIWebView を Outlet にしたもの。
        var scrollView = webView.ScrollView;

        // Bounces の影を消す via http://stackoverflow.com/questions/8480571/removing-shadows-from-uiwebview
        scrollView.Subviews.Where(v => v is UIImageView)
            .ToList().ForEach(v => v.Hidden = true);

        // 上に引っ張った時に見える背景とラベル
        var bounceBackground = new UIView(
            new RectangleF(0f, 0f, webView.Frame.Width, webView.Frame.Height));
        bounceBackground.BackgroundColor = UIColor.LightGray;
        var bounceLabel = new UILabel(
            new RectangleF(0f, webView.Frame.Height - 30f, webView.Frame.Width, 30f));
        bounceLabel.Text = "Pull up to Close";
        bounceLabel.TextAlignment = UITextAlignment.Center;
        bounceLabel.BackgroundColor = UIColor.Clear;
        bounceLabel.Opaque = false;

        // 背景とラベルを WebView の一番奥に追加する
        webView.InsertSubview(bounceLabel, 0);
        webView.InsertSubview(bounceBackground, 0);

        // 適当な URL を読み込み
        webView.LoadRequest(NSUrlRequest.FromUrl(new NSUrl("http://yahoo.co.jp/")));
		
        // 閉じるのに必要な分だけ上に引っ張ったら true になる
        var canClose = false;

        // ドラッグ開始時にフラグOFF(一応)
        scrollView.DraggingStarted += (sender, e) => 
        {
            canClose = false;
        };

        // ドラッグ終了時、必要量引っ張っていたら OnCloseByPullUp を呼ぶ
        scrollView.DraggingEnded += (sender, e) => 
        {
            if (canClose)
            {
                OnCloseByPullUp();
            }
        };

        // スクロールした時にいろいろやる
        scrollView.Scrolled += (sender, e) => 
        {
            var labelFrame = bounceLabel.Frame;

            // コンテンツの一番下まで表示してさらに引っ張ったサイズ
            var offsetY = (scrollView.Frame.Height + scrollView.ContentOffset.Y) 
                - scrollView.ContentSize.Height;

            // 50px 上に引っ張ったら閉じるものとする
            canClose = offsetY > 50f;
            bounceLabel.Text = canClose ? "Release to Close" : "Pull up to Close";

            // ラベルがいつまでも移動しないように
            if (offsetY > labelFrame.Height)
            {
                offsetY = labelFrame.Height;
            }

            // ラベルがドラッグと共に下からせり出してくるように
            labelFrame.Y = scrollView.Frame.Height - offsetY;
            bounceLabel.Frame = labelFrame;
        };
    }
    
    // "Release to Close" で離すと呼ばれる
    void OnCloseByPullUp()
    {
        var v = new UIAlertView("", "Close this view",  null, "Close");
        v.Show ();
    }
}
```

## やってる事

1.  ScrollView の「引っ張った時に見える場所（= Bounce というらしい）」の影を消す。 via http://stackoverflow.com/questions/8480571/removing-shadows-from-uiwebview
2. 背景と、ラベルを WebView 内の一番奥に挿入する（引っ張った時にのみ見えるように）
3. あとはイベントハンドラでの処理。スクロール中に、「最下部で引っ張り中」だったら "Pull up to Close" ラベルをアニメーションさせながら表示する。50px 以上引っ張ってたら "Release to Close" にラベルを変える。「閉じられるよ」フラグも ON にしとく。
4. ドラッグ終了イベントで、「閉じられるよ」フラグが立ってたら、コールバックする。

## 今後

もうちょっとライブラリっぽくしたいですね。あと引っ張り中にアイコンとか表示させたい。