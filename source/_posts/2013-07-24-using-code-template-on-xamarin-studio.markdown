---
layout: post
title: "Xamarin Studio でコードテンプレートを使う"
date: 2013-07-24 17:48
comments: true
categories: [Xamarin, MvvmCross]
---
Visual Studio や Eclipse とか、IDE ならだいたい備えているコードテンプレート、呼び名はそれぞれ違いますが、``for`` ってタイプすると ``for (object o : items) {  }`` 的なコードのひな形を生成してくれる機能の事です。
<!--more-->

Xamarin Studio にも当然ありまして、その使い方を説明します。

## きっかけ

MvvmCross っていうフレームワークの Tutorial 動画で Visual Studio を使っているんですが、その中で多用されてるので、真似してみたくなりました。

例えばこれ

* [N=0 : A first MvvmCross Application (N+1 days of MvvmCross) | N+1 days of MvvmCross](https://www.youtube.com/watch?v=_DHDMNB_IeY&list=PLR6WI6W1JdeYSXLbm58jwAKYT7RQR31-W&feature=player_detailpage&t=178)

## 手順

### 1. Xamarin Studio の Preference を開く

システムメニュー → Preference → テキストエディタ → コード テンプレート です。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/using_code_template_in_xamarin_studio_01.png)

###  2. コードテンプレートを作成する

追加 で "新しいテンプレート" の画面を開き、

![img2](https://dl.dropboxusercontent.com/u/264530/qiita/using_code_template_in_xamarin_studio_02.png)

のように設定します。

* ショートカット : pmvx
* グループ : C#
* 説明 : 適当に
* Mime : text/x-csharp
* [展開されるテンプレート] にチェックを入れる

テンプレート テキスト は以下の通り

```c#
private $type$ $property$;
public $type$ $Property$
{ 
	get { return $property$; }
	set { $property$ = value; RaisePropertyChanged(() => $Property$); }
}
```

``$type$``, ``$property$``, ``$Property$`` という3つの変数を使っています。画面右端にあるドロップダウンで、各変数が選択できるので、それぞれ Default で既定値を設定します。

ここでは、以下のようにしました。

* type : object
* property : _property
* Property : MyProperty

OK を押して保存します。

### 3.  使ってみる

コードエディタで ``pmvx`` とタイプすると、

![img2](https://dl.dropboxusercontent.com/u/264530/qiita/using_code_template_in_xamarin_studio_03.png)

となり、タブを2回ほど押すと、

![img2](https://dl.dropboxusercontent.com/u/264530/qiita/using_code_template_in_xamarin_studio_04.png)

と、テンプレートコードが挿入されます。
あとは、ハイライトされている部分を変更すると、テンプレート内も連動して変更されます。

これでコード入力が楽になりました。

コードテンプレートのインポート／エクスポートや、Visual Studio との互換性なども調べてみたいですね。
