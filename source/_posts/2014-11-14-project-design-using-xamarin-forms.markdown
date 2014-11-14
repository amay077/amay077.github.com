---
layout: post
title: "Xamarin.Forms をガチで使うときのプロジェクト構成案"
date: 2014-11-14 19:34:02 +0900
comments: true
categories: [Xamarin, Android, iOS]
---
　[Xamarin.Forms](http://xamarin.com/forms)（以下 XF とします）を本格的に開発に導入してみようと思ってるんだけど、その時のプロジェクト(アセンブリ)構成はこんな感じかな、というのを考えてみた。
<!--more-->
## コンセプト

* XF を使う理由は、Viewのワンソース化よりも *DataBinding機構が使えること*
* いざという時逃げられるように XF への依存を最小限にする
* iOS/Android 対応アプリ開発のこと、他は知らん
* MVVM で PCL で、PCL のところを最大化する

## プロジェクト構成図

　角丸四角形がプロジェクトを、矢印は依存を示す。

![](https://dl.dropboxusercontent.com/u/264530/qiita/project_design_using_xamarin_forms_01.png)

左から説明。

### XF.Android, XF.iOS, XF.Core

　Xamarin.Forms ソリューションを作成するとテンプレで作られるプロジェクト群。
　XF.Core でプラットフォームに依存しないView（ボタンとか）を、XF.Android/XF.iOS でプラットフォーム固有のView（スライドメニューとかNotificationなど）を提供する。
　これらは BindableProperty を介して ViewModel とデータバインディングする。
　XF は他に、ServiceLocator や MessageCenter を提供するが、それらは使用しない（ロックインを防ぐため）
　結局この案では、「XF=View層のみ」となる。

### App.Core

　アプリのView以外の共通部分のプロジェクト。ViewModel と Model を含む。
　ViewModel から View へシグナルを送るために "XFではない" Messengerを使う。（MvvmLight とか Prism とかから引っこ抜いてくればいいかな？）
　Model にはビジネスロジックのみを記述し、通信処理やデータI/Oなどのプラットフォーム共通なAPIはModelから直接使い、プラットフォーム固有の機能は、ServiceInterface を使う。
　ServiceInteface は、ServiceLocator によって App.Android/iOS から実体が Inject される。ServiceLocator は "XFではない"…以下略
　Rx を使うので、たぶん Model のメソッドの返り値は全部 IObservable<T> になります。

### App.Android, App.iOS

　プラットフォーム固有のAPI層。例えば GPS とか、アプリ連携とか、アイテム課金とか。ServiceInterface に定義されている Interface を実装するところ。

## 懸案

* PUSH通知受信とか、本来は Platform Specific APIs で担当したいが、プラットフォームの都合で、View で受信しなければならない機能の落としどころ。
* App.core をもっと分割した方がよい「ViewModelからAPI呼ばないよね？」とか「App.AndroidからModelにアクセスできるのがイヤ」とかを厳格に制限しようと思ったら分割した方が良さそう。
* Model から左側を全部 IObservable<T> 化しようと思っているが、Callback→Observable変換をModelでやるか、API層でやるか。"ビジネス"ロジックではないので、右側かな。
* XF.Core にどれだけ詰め込むか。画面レイアウトもXFでできるだけ頑張る、画面遷移フレームワークもXFで用意する、か？Sketches がどこまで活用できるか？

さて、どんなもんでしょ？
