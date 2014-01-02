---
layout: post
title: "Xamarin.iOS で ViewController の名称を変える"
date: 2013-12-13 0:00
comments: true
categories: [Xamarin, XAC13, iOS, C#]
---
今日も小ネタです。

Xamarin.iOS の SingleView Application とか(storyboard じゃないやつ)でプロジェクトを作ると、唯一の ViewController の名称が 「プロジェクト名ViewController.cs(.xib)」ってなるんですけど、なんかカッコ悪い。
<!--more-->
MainViewController とか、StartupViewController にしたいですね。その方法です。

## 1. プロジェクトを作る

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_ios_rename_viewcontoller_01.png)

「RenameTest」という名称でプロジェクトを作ると、ViewController は ``RenameTestViewController.cs`` になります。

## 2. クラス名をリファクタ機能で変更

``RenameTestViewController.cs`` を開いてクラス名のところで右クリック→リファクタ。

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_ios_rename_viewcontoller_02.png)

``MainViewController`` に変更します。

これにより以下のようにファイルが変更されます。(git status を晒すことでスペースを稼ごう…)

```
$ git status
 On branch master
 Changes to be committed:
	modified:   AppDelegate.cs
	renamed:    RenameTestViewController.cs -> MainViewController.cs
	renamed:    RenameTestViewController.designer.cs -> MainViewController.designer.cs
	modified:   RenameTest.csproj
```

## 3. xib ファイル名を変更する

``RenameTestViewController.xib`` はリファクタに追従しないので、手動でファイル名を変更します。

それから、``MainViewController`` のコンストラクタで、リテラルに xib の名称を指定しているのでそこも修正します。

```csharp MainViewController.cs
namespace RenameTest
{
    public partial class MainViewController : UIViewController
    {
        public MainViewController() : base(“MainViewController", null) // ← ココ！
        {
        }
```

## 4. Xcode との連携確認

.xib ファイルをダブルクリックすると Xcode が起動します。これは OK。けどファイル一覧を見てみると…

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_ios_rename_viewcontoller_03.png)

``RenameTestViewController.h`` ってファイルができてる！
``MainViewController.h`` になってほしいのですが。。。

Xamarin Studio にもどって、「RenameTestViewController」 が残っているところを探します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_ios_rename_viewcontoller_04.png)

お前らか！

``MainViewController.designer.cs`` は、マニュアルで修正してはいけないのですが、試しに直してみます。

``MainViewController.xib`` 内の方も古い名前は抹殺しておきます。
これは Xamarin Studio ではソースコードエディタで開くか、別なテキストエディタで編集します。(これも推奨されない)

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_ios_rename_viewcontoller_05.png)

どちらも直したら、再度 ``MainViewController.xib`` を実行して Xcode を起動。

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_ios_rename_viewcontoller_06.png)

やったー、ようやく ``RenameTestViewController`` を抹殺できました。この状態で Xamarin 側との outlet の連携など、問題ないようです。

まあ、Xcode 用のプロジェクトファイルは Xamarin Studio が自動生成するもの(obj ディレクトリに作成される)で、名前が元のままでも問題はないです。

以上、手順をまとめてみましたが、経験的に、些細な変更でアプリが起動できなくなったりするので、こまめにコミットしておく事をおすすめします。

Storyboard のプロジェクトだったら、[以前書いた](http://qiita.com/amay077/items/716742474bce343c5729)ように Xamarin Studio だけで完結できるので簡単ですが、まだα版です。はやくリリースされるといいですね。