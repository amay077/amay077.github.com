---
layout: post
title: "iOS アプリでアラート出してボタンが押されるまで待つ？方法を Objective-C と Xamarin.iOS で比べてみた"
date: 2014-03-10 22:09
comments: true
categories: [Xamarin, iOS, C#, Objective-C]
---
なんか割とニーズがあるみたいで。
<!--more-->
* [【Objective-C】アラート（UIAlertView）でボタンを押すまで次の処理を待つ方法 - creativi.tea](http://teapipin.blog10.fc2.com/blog-entry-224.html)
* [Cocoaの日々: [iOS] 非同期処理を同期処理に変える](http://cocoadays.blogspot.jp/2011/05/ios.html)
* [[Objective-C] UIAlertViewを同期処理する - Qiita ](http://qiita.com/edo_m18/items/cb1d9061d91e572b58eb)

## Objective-C の場合

``UIAlertView`` は、結果を受け取るのが deletgate で、 Objective-C では、(Blocks を使わなければ) 受け取りが別メソッドになってしまう、しかも複数のアラートの結果が同じメソッドに飛んでくるので、tag値で分岐…とかいろいろで、ホントに使うのが面倒ですね。

さらに、「アラートの結果を受け取ってから、次の処理を行う」という処理を素直に記述したいと思うと、上で示したような「アラートを表示して、結果が得られるまで while で待つ」というなんとも不格好なコードになってしまいます。下にも書いてみました。（あ、メンバ変数も使わざるを得ないし。）

```objective-c HogeViewController.m
@implementation HogeViewController {
    NSInteger _buttonIndex;
}

- (IBAction)buttonTouchUp:(id)sender
{
    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:nil
                                message:NSLocalizedString(@"なにか押して",@"")
                               delegate:self
                      cancelButtonTitle:@"Cancel"
                      otherButtonTitles:@"OK", nil];
    [alert show];
    
    // ボタンが押されるまで待つ
    _buttonIndex = -1;
    while (_buttonIndex == -1) {
        [[NSRunLoop currentRunLoop]
         runUntilDate:[NSDate dateWithTimeIntervalSinceNow:0.5f]]; // 0.5秒
    }
    
    label1.text = [NSString stringWithFormat:@"%d 番目のボタンを押したね", (int)_buttonIndex];
}

-(void)alertView:(UIAlertView*)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
	_buttonIndex = buttonIndex;
}
```

``[NSRunLoop runUntilDate]`` なんて VB6 の ``DoEvents`` ですもんねなつかしい。

## Xamarin.iOS(C#) の場合

Objective-C ではややこしかった「アラートを出す→ボタンを押す→次の処理へ」という流れ、 Xamarin.iOS と C# ならこんなにシンプルに書けます。

```csharp HogeViewController.cs
async void OnButtonTouch(object sender, EventArgs e)
{
    var buttonIndex = await MsgBox("", "なにか押して", "Cancel", "OK");
    label1.Text = buttonIndex.ToString() + "番目を押したね";  
}

static Task<int> MsgBox(string title, string message, 
    string cancelButtonTitle, params string[] buttons)
{
    var comp = new TaskCompletionSource<int>();
    
    var alert = new UIAlertView(title, message, null, cancelButtonTitle, buttons);
    alert.Clicked += (_, e) => comp.TrySetResult(e.ButtonIndex);
    alert.Show();
    
    return comp.Task;
}
```

VB6 が懐かしくて ``MsgBox`` ってメソッドにしちゃいましたよ。

``MsgBox`` は、Task を返す **非同期な** メソッドです。この非同期処理が終了するのは、``TaskCompletionSource.TrySetResult`` が呼び出された時、つまりアラートのボタンが押された時です。この非同期処理の戻り値はもちろん押したボタンのインデックスです。

非同期メソッドである ``MsgBox`` を呼び出す側には、キーワード ``await`` が付いています。
これをつけると、次行以降の処理は、非同期の MsgBox が完了した後、順次実行されます、つまり待っているわけではなく、どちらかというと、 **処理を後続に付け足す** 感じ。
さらに、この後続処理はUIスレッドで実行されるので、UIパーツへのアクセスも問題ありません。

``async`` はメソッド内で ``await`` を使うときにつけるお約束。

async/await は一見、ただの同期処理に見えるので理解して使う必要がありますが、Objective-C のコードに比べて、とても簡潔に、流れるように書くことができるのが分かると思います。

**C# の非常に強力な言語機能は、[Xamarin](https://xamarin.com/) を選択する大きな理由の一つです。**

## 参考

* [c# - Messagebox.Show and DialogResult equivalent in MonoTouch - Stack Overflow](http://stackoverflow.com/questions/4613071/messagebox-show-and-dialogresult-equivalent-in-monotouch)
* [async/awaitと同時実行制御 | ++C++; // 未確認飛行 C ブログ](http://ufcpp.wordpress.com/2012/11/12/asyncawait%e3%81%a8%e5%90%8c%e6%99%82%e5%ae%9f%e8%a1%8c%e5%88%b6%e5%be%a1/)
* [async/await不要論](http://www.slideshare.net/bleistift/asyncawait2)