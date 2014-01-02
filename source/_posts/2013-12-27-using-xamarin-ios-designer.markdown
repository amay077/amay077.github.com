---
layout: post
title: "Visual Studio なんて不要？Xamarin Studio の iOS用UIデザイナを試す！"
date: 2013-12-9 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#]
---
Build Insider さんでも Xamarin の記事が公開され、影響力では完全に喰われてしまっている [Xamarin Advent Calendar 2013](http://qiita.com/advent-calendar/2013/xamarin) です。
<!--more-->
* [Visual StudioでiOS／Androidアプリが書けるXamarinを試してみた（iOS編） - Build Insider](http://www.buildinsider.net/mobile/xamarinvisualstudio/01)

> 通常、Mac上のXcodeでiOSアプリのUIを作成する場合には、
>
>　1.「Interface Builder」というGUIデザイナーを使う
>　2. コードでUIを記述する
>
>という2種類の手段がある。
>
> 細かく表示位置などを制御したい場合などでは2.の方が好まれる傾向にある。

> Xamarin for Visual Studioでも、この2.の方法で作成することになる（逆にいうと、Interface Builderはサポートされていないので、コードでしか作成できない）

え、そうなんですか。個人的にはコードでUIを記述するのは破滅への入り口って思ってるんですが…。

Mac で動かす Xamarin Studio では、

1.「Interface Builder」というGUIデザイナーを使う
2. コードでUIを記述する

どちらも使うことができます。以下参照で。

* [Xamarin.iOS の Tutorial をなんとなくトレースしてみる - Qiita [キータ]](http://qiita.com/amay077/items/bac6621007ecfe7dddb9)

さらに、目下開発中の Xamarin Studio では、iOS デザイナを搭載しており InterfaceBuilder すら必要無くなります。

今回は、 Xamarin Studio の α版 を使って、iOS用デザイナを使ってみます。

(Visual Studio で開発しても、iOSアプリのデバッグやビルドには Mac が要るのだから、初めから Mac の Xamarin Studio で開発しようぜ、という狙いです。)

## 1. Xamarin Studio を α版 に切り替える

Xamarin Studio を起動したら、システムメニューの 「Check for Updates...」で channel を 「Alpha」 に switch 、インストール後 Restart します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_01.png)
![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_02.png)

## 2. プロジェクトを作成する

新しいプロジェクト → iOS → iPhone Storyboard → Single View Application で名前は「UIDesignerTest」として OK します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_04.png)

## 3. UIデザイナを開く

左側のビューにある ``MainStoryboard.storyboard`` をダブルクリックすると、中央にUIデザイナが開かれます。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_05.png)

Interface Builder とそっくりでしょう？

## 4. UIデザイナだけで画面遷移を実装してみる

まったくコーディングせずに、画面遷移してみます。

右側ビューにある 「Toolbox」から「Navigation Controller」をデザイナにドラッグ＆ドロップします。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_06.png)

ちょっと見やすいようにレイアウトを整えてみます。(中央右上の +/- ボタンで拡大/縮小ができます)

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_07.png)

画面遷移の線を Navigation Controller → ViewController(元々あったやつ)  につなぎ直します。Action を尋ねられるたら「Push」を選択します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_08.png)

Toolbox から Button をドラッグ＆ドロップで配置します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_09.png)

Button を Ctrl キーを押しながらドラッグして、もう一つの ViewController でドロップします。Action を尋ねられるので「Push」を選択します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_10.png)

はい、できあがりです。実行してみます。
iOS Simulator から任意のデバイスを選んで「実行」します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_12.png)

こんな感じです。Storyboard ライクに画面遷移が実装できました。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_13.gif)

## 5. ボタンの処理などを実装してみる

今度はボタンの処理などを実装してみます。

右端の ViewController に、TextField, Button, Label をそれぞれドラッグ＆ドロップして配置します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_14.png)

ViewContorller のコードビハインド(っていうのか？)を作成します。
ViewController の下の黒いところを選択して、プロパティビュー(表示されてない場合は、メニュー→ビュー→パッド→プロパティ)の Class に ``DetailViewController`` と入力して Enter します。すると DetailViewController.cs などが作成されます。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_15.png)

次に、Label や Button などをコード上の変数にします。(iOS の世界では Outlet って言います)
TextField を選択して、プロパティビューの Name に ``text1 `` と入力して Enter します。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_16.png)

すると、``DetailViewController.designer.cs`` にプロパティ ``text1`` が作成されています。

```csharp DetailViewController.designer.cs
using MonoTouch.Foundation;
using System.CodeDom.Compiler;

namespace UIDesignerTest
{
	[Register ("DetailViewController")]
	partial class DetailViewController
	{
		[Outlet]
		[GeneratedCodeAttribute ("iOS Designer", "1.0")]
		MonoTouch.UIKit.UITextField text1 { get; set; }
		
		void ReleaseDesignerOutlets ()
		{
			if (text1 != null) {
				text1.Dispose ();
				text1 = null;
			}
		}
	}
}
```

同じ手順で Label からプロパティ ``label1`` を作成します。

次に、ボタンが押された時に、text1 の内容を label1 に表示するようにします。

Button の TouchUpInside イベントにハンドラを作ります。(iOS的には Action といいます。)

Button を選択して、プロパティビューを表示し、上部にある「Event」を選択します。しばらく待つと Control Events が表示されるので、Touch → Up Inside の項目に ``button1_TouchUpInside`` と入力して Enter します。(メソッド名は Visual Studio の慣例？に従ってますが、任意の名称で OK です。)

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_17.png)

すると、コードエディタが開かれてどこに Add Event Handler するか聞かれるので、適当な位置で Enter すると、 ``button1_TouchUpInside`` のコードが生成されます。

そこに、TextField の内容を Label に表示するコードを書きましょう。

```csharp DetailViewController.cs
namespace UIDesignerTest
{
    public partial class DetailViewController : UIViewController
    {
        public DetailViewController (IntPtr handle) : base (handle)
        {
        }

        partial void button1_TouchUpInside(UIButton sender)
        {
            label1.Text = text1.Text; // 記述したコード
        }
    }
}
```

これで完成。動かしてみます。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_xamarin_ios_builtin_designer_18.gif)


## まとめ

Xamarin Studio の強力な iOS UIデザイナを紹介しました。
現在 Stable な Xamarin Studio にはこれは搭載されていませんが、α版に切り替えることで今すぐ試すことができます。

Visual Studio の Addin にも搭載されるようですが、今それが使えるかは分かりません。

冒頭で述べた通り、Visual Studio 使わずに Xamarin Studio だけでも十分実用レベルなので、最初は Xamarin Studio で試してみてもよいと思います。

## 参考

* [Evolve 2013 Conference – Xamarin](http://xamarin.com/evolve/2013) の 「iOS Designer」のチャプターから。歓声がすごいｗ
* [Designer | Xamarin](http://docs.xamarin.com/guides/ios/user_interface/designer/)
* [Hands-on: Xamarin Studio’s powerful new iOS designer | Xamarin Blog](http://blog.xamarin.com/hands-on-xamarin-studio%e2%80%99s-powerful-new-ios-designer/)