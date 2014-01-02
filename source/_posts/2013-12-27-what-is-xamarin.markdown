---
layout: post
title: "Xamarin(ザマリン) とはなんぞや"
date: 2013-12-01 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#]
---

[Xamarin Advent Calendar 2013](http://qiita.com/advent-calendar/2013/xamarin) 始まりました。

先は長いので、初日は Xamarin(ザマリンと読みます) とはなんぞや、個人開発者として使う時にどうなるの、的な事をさらっと書いてみようと思います。
<!--more-->
Xamarin 自体は企業名であり、その歴史は .NET の Linux 版を開発していた Ximian という企業が Novell に買収されて、その後レイオフされて作った企業です。

* [Xamarin - Wikipedia](http://ja.wikipedia.org/wiki/Xamarin)
* [Xamarin - Official site](http://xamarin.com/)

で、同社が開発した、 .NET技術で iOS や Android アプリが作成できる SDK が、Xamarin.iOS だったり、Xamarin.Android だったりするわけですが、それらに Mac アプリを開発できる Xamarin.Mac や、Xamarin Studio という統合開発環境を加えたツール群をまるごとひっくるめて Xamarin と呼んでいます。

## Mono, iOS, Android, Mac アプリ、.NET でも使える DLL が作れますよ

.NET の OSS 実装である Mono を利用したソフトウェアは無償で作成できます。
Xamarin.Android/iOS/Mac による各PF用のアプリが作成できます。

Windows 用の .NETアプリ(Windows.Forms や WPF)は、Xamarin 自体では作成できません。それは Visual Studio の役割です。ただ、PCL(Portable Class Library)と呼ばれる、PFを問わず動作するアセンブリ(DLL) を作成できますし、そもそも Mono と .NET の API はほとんど同じなので、書いたコードは Windows でも流用できます。

## クロスプラットフォームで共通にできるのは「コア」な部分だけですよ

いわゆる幻想を壊しておきます。

Xamarin.XXX が提供するのは、

**各PF版の.NET API ＋ 各PFのAPIの.NETラッパクラス**

です。

「各PF版の.NET API」とは、いわゆる基本クラスで、基本的な型だったり、文字列処理だったり LINQ だったりその他もろもろです。

一方、「各PFのAPIの.NETラッパクラス」とは、Android なら Android SDK、iOS なら CocoaTouch の API を .NET で記述できるラッパーです。ここにPF間の互換性はありません。

なので、画面を作るのに Xamarin.Android なら ``Activity`` クラスを使いますし、Xamarin.iOS なら ``ViewController`` クラスを使います。
GPS を使うのに、.Android なら ``LocationManager`` を使いますし、.iOS なら ``CLLocationManager`` を使います。

つまり、「画面」と「各PF固有の機能」は共通化することができません。従って、各PF の API は理解しておく必要があります。それから .NET Framework の基本クラスライブラリも。

上司に言うと「なんだその程度か」と返されると思いますが、 **Objective-C を書かないで良い** という事だけで十分に価値があると思うんですよね僕は。(お宅の(SI)会社に iOS技術者何名居ます？)

ほかのクロスプラットフォーム開発可能な SDK（Titanium とか Abobe AIR とか）との比較は後日書こうと思います。

## Xamarin Studio もしくは Visual Studio で開発しますよ

Mac の人は Xamarin Studio 一択、Win な人は Xamarin Studio もしくは Xamarin のアドインにより Visual Studio で開発ができます。
ただし BUSINESS エディション以上が必要です。

また iOS アプリを開発する場合は ビルドやデバッグのために Mac が必要なので、現実的には Mac 必須という感じです。

なら Xamarin Studio に慣れようよ、というのが個人的なスタンスですが、Visual Studio 強力ですもんねぇ…。

## プロプライエタリな、有償の製品ですよ

Mono はオープンソースですが、 Xamarin.Android、 Xamarin.iOS、Xamarin.Mac は有償プロダクトです。
.Android と .iOS はソースも公開されていません。
.Mac については、「MonoMac というOSSのライセンスが GPL/LGPL なため用意された商用ライセンスである」と説明されています。

* [MonoMac - MonoBook](http://monobook.org/wiki/MonoMac#.E3.83.A9.E3.82.A4.E3.82.BB.E3.83.B3.E3.82.B9)

Xamarin Studio はオープンソースな IDE です。(元は MonoDevelop) なので、純粋な Mono アプリケーションを作成するのにも使用されます。Unity での開発にも使えるみたいです（詳しくないですが…）

* [UnityのためにXamarin Studioのセットアップ - けいごのなんとか](http://anchan828.hatenablog.jp/entry/2013/02/21/160836)

## 価格 - 開発者毎、プラットフォーム毎、年毎に料金頂きますよ

まあ、個人では高い部類に入ると思います。企業ユースでは、先日発表された Microsoft との提携により実現されたサブスクリプションがありますが、それはまた別の機会に。

* [Store - Xamarin](https://store.xamarin.com/)

SDK をダウンロードすると、まずは無償の STARTER エディションから始まりますが、これは作成できるアプリケーションのバイナリサイズに限界があり、ちょっと外部ライブラリを使おうとするとすぐに制限を超えます。私は Json.NET を使おうとしたら超えました。つまり実質無意味です。

次に BUSINESS エディションのトライアル(30日間)が始まります。この時は Xamarin へのユーザー登録が必要です(要メアド)。このトライアル期間中に作成したアプリは、ビルドから24時間以内しか動作しない時間制限付きです。しかし機能的な制約はありません。
また前述のとおり、 Visual Studio での開発をしたい場合は、このエディションが必要です。

もっとも注意すべき点は、価格は、 **「プラットフォーム毎」の年間ライセンス** だということです。
うっかり 「INDIE エディションで $299 だせば、iOS/Android/Mac アプリ作って公開できるぜ〜」と勘違いしてしまうと後で激しく落胆します。(誰？

### 学割があるよ

* [Do you have any student or academic pricing? Frequently Asked Questions - Xamarin](http://xamarin.com/faq#q32) 

にて、アカデミックライセンスについて **「BUSINESS エディションを $99/PF/Year で提供」** と記述されています。

日本では エクセルソフトさんが、窓口になってくれるそうです。

* [Xamarin 価格 - シンプルで分かりやすい価格体系 : XLsoft エクセルソフト] (http://www.xlsoft.com/jp/products/xamarin/price.html) - アカデミック割引はありますか？

エクセルソフトの中の人によりますと、

<blockquote class="twitter-tweet" data-conversation="none" lang="ja"><p>. <a href="https://twitter.com/amay077">@amay077</a> こんにちは。Xamarin の学割販売、できますよ。 <a href="http://t.co/yM1xHEi8A0">http://t.co/yM1xHEi8A0</a> の アカデミック割引はありますか？をご覧ください。確認事項が少々あるのと、ドル建てより割高感が凄いかもしれませんが、2万円弱でサポートなしです。</p>&mdash; 田淵 義人 (@ytabuchi) <a href="https://twitter.com/ytabuchi/statuses/405997166352543744">2013, 11月 28</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

との事です。

## 日本語情報について

公式サイトの日本語化については、前出のエクセルソフトさんが鋭意翻訳中とのことです。
エクセルソフトの中の人である [@ytabuchi](https://twitter.com/ytabuchi) さんに希望を伝えると、その箇所の翻訳が優先されるかもしれないとのことでした。

* [Xamarin (ザマリン) ホーム : XLsoft エクセルソフト](http://www.xlsoft.com/jp/products/xamarin/index.html)
* [コラム/C#で作れる！ iOS & Android クロス開発環境 Xamarin を試す - WisdomSoft](http://www.wisdomsoft.jp/615.html)
* [ものがたり](http://atsushieno.hatenablog.com/) - Xamarin の中の人である [@atsushieno](https://twitter.com/atsushieno) 氏のブログ

あとこちら、Qiita の Xamarin タグにも、勝手ながら私めが頑張って連投しております。

* [Xamarinに関する新着投稿 - Qiita](http://qiita.com/tags/xamarin/items)

Microsoft との提携をきっかけに、日本でも Xamarin が流行ってくれるといいなと期待しています。

Advent Calendar 初日はこんなもんで。