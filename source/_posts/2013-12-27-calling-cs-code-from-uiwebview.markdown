---
layout: post
title: "Xamarin.iOS で UIWebView から C# のコードを呼び出す方法"
date: 2013-12-20 0:00
comments: true
categories: [Xamarin, XAC13, iOS, C#]
---
まあ、

* [WebView で Javascript と Objective-C のコードを相互に呼び出す方法 | 【スマホ×HTML5】Web&ハイブリッドアプリ開発者ブログ](http://i26.jp/html5dev/webview-%e3%81%a7-javascript-%e3%81%a8-objective-c-%e3%81%ae%e3%82%b3%e3%83%bc%e3%83%89%e3%82%92%e7%9b%b8%e4%ba%92%e3%81%ab%e5%91%bc%e3%81%b3%e5%87%ba%e3%81%99%e6%96%b9%e6%b3%95/)

を Xamarin.iOS でやってみたという話なだけです。
<!--more-->
## サンプル

```csharp MainViewController.cs

/* in MainViewController.designer.cs
[Outlet]
[GeneratedCodeAttribute ("iOS Designer", "1.0")]
MonoTouch.UIKit.UIWebView MyWebView { get; set; }
*/

public override void ViewDidLoad()
{
    base.ViewDidLoad();
    
    MyWebView.LoadRequest(NSUrlRequest.FromUrl(NSUrl.FromString("http://www.hatena.ne.jp/")));

    // LoadFinished はイベント
    MyWebView.LoadFinished += (sender, e) => {
        if (String.Equals(MyWebView.Request.Url.Host, "b.hatena.ne.jp"))
        {
            new UIAlertView("Load Finished", "ページが表示されました", null, "Close").Show();
        }
    };

    // ShouldStartLoad は Delegate
    MyWebView.ShouldStartLoad = (webView, req, navType) =>
    {
        var permited = !String.Equals(req.Url.Host, "hatenablog.com");
        if (!permited)
        {
            new UIAlertView("ShouldStartLoad", “はてなブログは見ちゃダメ！", null, "Close").Show();
        } 

        return permited;
    };
}
```

画面に UIWebView を一つ貼り付けて、 www.hatena.ne.jp を表示してます。

はてなブックマーク(b.hatena.ne.jp) へ移動すると、ページ読み込み後に「ページが表示されました」とポップアップします。
はてなブログ(hatenablog.com) へ移動しようとすると、「はてなブログは見ちゃダメ！」とポップアップし、移動はキャンセルされます。

Objective-C の、``webViewDidFinishLoad`` , ``shouldStartLoadWithRequest`` デリゲートが、Xamarin.iOS では、 ``LoadFinished`` , ``ShouldStartLoad`` に対応します。

注意点は、``LoadFinished`` はイベントであるのに、 ``ShouldStartLoad`` は delegate であるという事です。

Obj-C のデリゲートは、Xamarin.iOS では全てイベントになっているのかなーと思っていましたが、 ``ShouldStartLoad`` のように「値を返す」必要があるものについては delegate になっているようです。

Xamarin.iOS の APIデザインについては、公式サイトに説明があります。

* [API Design | Xamarin](http://docs.xamarin.com/guides/ios/advanced_topics/api_design/)

ガッツリ熟読した方がよさそうですねえ。

最後に、作ったサンプルを動かしてみます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/calling_csharp_from_webview_01.gif)

なんか ``LoadFinished`` が２回呼ばれてる。重複チェックしなきゃダメですね
