---
layout: post
title: "Xamarin.iOS の Tutorial をなんとなくトレースしてみる"
date: 2013-03-14 23:07
comments: true
categories: [Xamarin, iOS, C#]
---

Xamarin.Android ネタばかり書いてきましたが、そろそろクロスプラットフォームのことも考えたいので Xamarin.iOS にも手を出してみたいと思います。ちなみに当方、iOS開発についてはシロートに毛が生えた程度です。

<!--more-->

Xamarin の公式チュートリアルがあるので、それを英語も読まずに雰囲気でトレースしてみたいと思います。

* [Hello, iPhone | A First Xamarin.iOS Application Getting Started - Tutorial 3](http://docs.xamarin.com/guides/ios/getting_started/hello%2C_world)

## プロジェクトの作成(Creating a new Xamarin.iOS iPhone Project)
とりあえず Xamarin Studio をたちあげて、新規 → ソリューション、C# → iOS → iPhone → Single View Application を選択、ソリューション名は「HelloWorld_iPhone」としました。

## 作成されたファイルを眺めてみる
!["create_solution"](https://dl.dropbox.com/u/264530/qiita/xamarin_ios_created_screen.png)

ソリューションツリーを眺めてみます。

[参照] の中には ``monotouch`` が見えますね。Xamarin.Android では ``Mono.Android`` がありました。クロスプラットフォームにするときにはこいつらに依存しない Portable なライブラリプロジェクトが必要そうです。

``AppDelegate`` とか ``xxxViewController`` とか iOS っぽいです、が ``.cs`` です。Obc-C のキモい構文じゃないだけでも嬉しいものです。

## Introduction to Xcode 4 and Interface Developer
Tutorial(英語)では突然 Xcode の説明が始まります、なんのこっちゃ。
これはえーと *.xib ファイルをダブルクリックすると Xcode が起動して UIレイアウトは Xcode の Interface Builder(IB) でやれよ、的な感じだからですねたぶん。
とりあえず ``HelloWorld_iPhoneViewController.xib`` をダブルクリックして IB を起動して追従します。

##UI レイアウトの作成 (Creating the Interface)
見よう見まねで、Tutorial と同じようにレイアウトしました。

!["ib1"](https://dl.dropbox.com/u/264530/qiita/xamarin_ios_ib_1.png)

## アウトレットとアクションを追加する (Adding Outlets and Actions to the UI)
だんだん和訳にすらならなくなって来てますが。。。
Outlets とは、「UI要素と関連づいた変数」、Actions は「イベントハンドラ」の意ですかね。

さて、Xcode のバーを展開して、``HelloWorld_iPhoneViewController.h`` を表示、それを ``option`` キーを押しながらクリックします。
するとウィンドウが縦に分割されて、右側にソースコードが表示されます。

!["ib1"](https://dl.dropbox.com/u/264530/qiita/xamarin_ios_ib_2.png)

### アウトレットの追加(Adding an Outlet)
[Click Me] と書かれたボタンを、``control``キーを押しながら、隣のソースコードの @end の上らへんにドラッグ＆ドロップします。
!["ib1"](https://dl.dropbox.com/u/264530/qiita/xamarin_ios_ib_3.png)
次に表示されるウィンドウで Name を **btnClickMe** と入力して [Connect] ボタンを押すと、、、

!["ib1"](https://dl.dropbox.com/u/264530/qiita/xamarin_ios_ib_4.png)

```objc
@property (retain, nonatomic) IBOutlet UIButton *btnClickMe;
```
という行が追加されました。
ここで一旦 Xcode を保存して、Xamarin Studio に戻り(Xcodeは終了しなくてOK)、``HelloWorld_iPhoneViewController.designer.cs`` を開いてみてみると、、、

```c# HelloWorld_iPhoneViewController.designer.cs
using MonoTouch.Foundation;

namespace HelloWorld_iPhone
{
	[Register ("HelloWorld_iPhoneViewController")]
	partial class HelloWorld_iPhoneViewController
	{
		[Outlet]
		MonoTouch.UIKit.UIButton btnClickMe { get; set; }
		
		void ReleaseDesignerOutlets ()
		{
			if (btnClickMe != null) {
				btnClickMe.Dispose ();
				btnClickMe = null;
			}
		}
	}
}
```

``btnClickMe`` がこっちにも追加されてる！なにこれすごい。

再び Xcode へ、btnClickMe と同じ要領で、次は Label もドラッグ＆ドロップし、Name は **lblOutput** とします。 

### アクションの追加(Adding an Action)
次はアクションを追加してみます。
[Action1] と書かれたボタンを ``control`` を押しながらD&D、次のウィンドウで、Connection を **Action** に変更、Name を **actnButtonClick** とします。

!["ib1"](https://dl.dropbox.com/u/264530/qiita/xamarin_ios_ib_5.png)

```objc
- (IBAction)actnButtonClick:(id)sender;
```
という行が追加されました。

次の [Action2] は、Tutorial には、Action1 と同じアクションに関連付けられるよ的な事が書いてあるぽいですが、面倒なので省略。

## コードを書く(Writing the Code)
またまた Xcode を保存、Xamarin Studio へ。(Xcode はこれでお役御免)

``HelloWorld_iPhoneViewController.designer.cs`` をみてみると、**lblOutput** と **actnButtonClick** に関連するコードが増えてるのが確認できます。

これは Partial クラスですから、もう ``HelloWorld_iPhoneViewController.cs`` でもボタンやアクションが使えるはずです。早速実装してみます。

```c# HelloWorld_iPhoneViewController.cs
namespace HelloWorld_iPhone
{
    public partial class HelloWorld_iPhoneViewController : UIViewController
    {
        protected int _numberOfTimesClicked = 0;

        <省略>
        		
        public override void ViewDidLoad()
        {
            base.ViewDidLoad();
			
            // Perform any additional setup after loading the view, typically from a nib.
            btnClickMe.TouchUpInside += (sender, e) => 
            {
                _numberOfTimesClicked++;
                lblOutput.Text = "Clicked [" +
                    _numberOfTimesClicked.ToString() + "] times!";
            };
        }
		
        <省略>

        partial void actnButtonClick(NSObject sender)
        {
            lblOutput.Text = "Action button " +  ((UIButton)sender).CurrentTitle + " clicked.";
        }
    }
}
```  

実装できたら、動かしてみます。

!["ib1"](https://dl.dropbox.com/u/264530/qiita/xamarin_ios_created_debugger_scceeded.png)

動いた！(Action2 は実装してないので動きません)

## まとめ
* Xamarin.iOS では、Xcode の IB で UIレイアウトを作成する
* ので Mac 必須(なんですか？)
* イベントハンドラの実装はラムダ式や Binding で行うことを考えると、Action って要らなくね？
* Xcode ワカラン

## Next
* プロジェクト作る画面に **iPhone Storyboard** とかあったので、ストーリーボード系はそっちでイケるみたいです。

## 参考
* [Xcode 4: Keyboard shortcut for switching Assistant Editor to Tracking (Automatic) mode? - StackOverflow](http://stackoverflow.com/questions/6710957/xcode-4-keyboard-shortcut-for-switching-assistant-editor-to-tracking-automatic#)