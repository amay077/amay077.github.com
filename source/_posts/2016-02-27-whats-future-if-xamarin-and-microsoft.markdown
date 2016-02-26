---
layout: post
title: "Xamarin が Microsoft に買収されたので、今後を勝手に予想"
date: 2016-02-26 23:59:59 +0900
comments: true
categories: [Xamarin,C#,Android,iOS]
---

いやー起きたら驚きました。いつも午前中は仕事があまり捗らないのですが(ぉぃ、今日はさらに手につきませんでした。
<!--more-->

* [A Xamarin + Microsoft Future | Xamarin Blog](https://blog.xamarin.com/a-xamarin-microsoft-future/)
* [Microsoft to acquire Xamarin and empower more developers to build apps on any device - The Official Microsoft Blog](http://blogs.microsoft.com/blog/2016/02/24/microsoft-to-acquire-xamarin-and-empower-more-developers-to-build-apps-on-any-device/)
* [【速報】Xamarin が Microsoft に買収されました - Xamarin 日本語情報](http://ytabuchi.hatenablog.com/entry/2016/02/25/084553)

続報は、 [3/30-4/1 の Build 2016](http://build.microsoft.com/) と [Xamarin Evolve 2016](https://evolve.xamarin.com/) を待てとのことですが、企業としての Xamarin は、様々な製品やツールを展開していますので、MS買収によって、それらが今後どうなるのか、勝手に予想してみます。（個人の主観と希望を多分に含みます。 ○とか△は、今後の継続性(MSから見たら買収効果)を示してます）

## [Xamarin.Android, Xamarin.iOS](https://xamarin.com/platform) → ◎

　Android SDK や iOS SDK(CocoaTouch) を C# から呼び出す、現在主力のプロダクト。
　
　Microsoft とのパートナーシップにより、すでに Visual Studio に半ば組み込まれていますが、それがさらに推し進められて、完全統合（追加インストールなし）されると思われます。

　完全統合されたからと言って、 iOSアプリをデバッグ、デプロイするには Mac が必要なわけで、厳密には「Visual Studio だけで開発できる」ことにはならないと思います。

　Xamarin という「ブランド」は、いつか表示されなくなってしまうのかも知れません。。。


## [Xamarin.Mac](https://xamarin.com/platform#desktop) → △

　C# で Mac のデスクトップアプリが開発できるというプロダクト。
　現在も、お世辞にも「存在感がある」とは言えないプロダクトで、買収によって今後が不安なもののひとつ。

　Microsoft も、ここに投資するメリットはあまり感じてないのではないでしょうか？
　OSS として公開、のような可能性があるなら嬉しいかも。


## [Xamarin.Forms](https://xamarin.com/forms) → ◎

　Xamarin.Android/iOS とは異なり、「単一のコードで複数のプラットフォーム向けのアプリを開発できる」プロダクト。

　画面は XAML(と言っても WPF とは異なる)で記述し、Android/iOS/Windows(UWP) の ``Activity/ViewController/Window`` は、``Page`` というクラスに抽象化されます。

　登場以来 Xamarin が最も注力してきたプロダクトで、Microsoft のマルチデバイス戦略にもフィットします(説明しやすいし、デモ受けもしやすいしね)。

　長らくUIエディタがない状態が続いていますが、買収によりいよいよ？ [Build 2016](https://build.microsoft.com/) と [Evolve 2016](https://evolve.xamarin.com/) が楽しみです。


## [Xamarin Studio](https://xamarin.com/studio) → ○

　[MonoDevelop](http://www.monodevelop.com/) という OSS の統合開発環境に Xamarin プロダクト向けの Addin を加えたもの。

　Windows では、あえてこれを使用する必要は無いに等しいですが、Mac では、重要なIDEになります。
　実際、 「Android と iOS アプリだけ」を開発する場合は、Mac の方が何かと都合が良いわけで、 **Mac + Xamarin Studio がベストチョイス** なわけです。

　[Roslyn 対応](https://developer.xamarin.com/releases/studio/xamarin.studio_6.0/xamarin.studio_6.0/) も進んでいるし、ほとんどは OSS だし、非Windows開発者向けのIDEを引っ込めるメリットは Microsoft にはないでしょう。（Windows版の Xamarin Studio は微妙かも）

　(遠い)将来的には、[Visual Studio Code](https://www.visualstudio.com/ja-jp/products/code-vs.aspx)からの流れで、 Visual Studio のようなものが Mac に登場すると良いなあ、と思います。

## [Xamarin Components](https://components.xamarin.com/) → ×

　Xamarin で使えるライブラリを有償/無償で公開できるストアなんですが、Xamarin が [nuget](http://www.atmarkit.co.jp/fdotnet/chushin/nuget_01/nuget_01_01.html) に対応して以来、徐々に影が薄くなり、同じライブラリでも nuget の方が新しい、なんてこともザラになってきました。

　「ライブラリを販売できる」というエコシステムも機能している感じがしないので、徐々になくなっていくのではないでしょうか（誰か困る人いるんだろうか？）。

## [Xamarin Test cloud](https://xamarin.com/test-cloud) → ◎

　クラウド上に実際のAndroid/iOSデバイスが用意されており、それを使用してテストが行える「デバイスファーム」としてのサービス、それから、[Carabash](https://developer.xamarin.com/guides/testcloud/calabash/introduction-to-calabash/) という自動テスティングフレームワークを指します。

　元々は [LessPainful という企業が提供していたサービスを Xamarin が買収した](http://techcrunch.com/2013/04/16/xamarin-launches-test-cloud-automated-mobile-ui-testing-platform-acquires-mobile-test-company-lesspainful/) したものです。

　[Amazon](https://aws.amazon.com/jp/device-farm/) や [Google](https://developers.google.com/cloud-test-lab/) もデバイスファームをサービスしているのに対し、 Azure はまだないようなので、これは Microsoft にとってメリット大だと思います。

　お値段高めで知られる同サービスなので、今後の値付けが気になります。
　
## [Xamarin Insights](https://xamarin.com/insights) → △

　[Crashlytics](https://try.crashlytics.com/) のようなクラッシュログ収集・解析サービスです。
　なんだか、 Microsoft には [Visual Studio Application Insights](https://azure.microsoft.com/ja-jp/services/application-insights/) というサービスがプレビュー版で出ているようで、丸かぶりですね。

　独立させて残す意味はあまりないので、 Application Insights に統合されていくのかなあ、という感じです。（実戦投入してるので、ちょっとどうしようかな…）

## [RoboVM](https://robovm.com/) → ×

　Java で iOS アプリが開発できる(CocoaTouch がよびだせる)という、まるで Xamarin のような製品だなあと思っていたら、実際に [Xamarin が買収してしまった](https://xamarin.com/pr/xamarin-acquires-robovm) プロダクト。

　買収以来特に動きもなく Xamarin の製品ラインナップに載ることもなく「？」な状態が続いていました。

　そんな感じで、さらに Microsoft が Java を推すか？…可能性は低いと思います。

## その他

### [Xamarin Android Player(Preview)](https://developer.xamarin.com/guides/android/getting_started/installation/android-player/) → △

　Xamarin 社が提供する高速Androidエミュレータ。

　Microsoft は [Visual Studio Emulator for Android](https://www.visualstudio.com/ja-jp/features/msft-android-emulator-vs.aspx) を持っていますからこれも丸かぶり。しかも VSエミュの方が多機能じゃないかな。

 　唯一、Mac向けには残すかも知れませんね。

### [Xamarin Profiler(Preview)](https://xamarin.com/profiler) → ○

　Xcode の Instruments みたいなのを作っちゃいました、というもの(今は Androidアプリのみ対応)。

　既に Visual Studio とも連携してるみたいだし、これは継続進化でしょう。

### [Xamarin Inspector(Early Preview)](https://developer.xamarin.com/guides/cross-platform/inspector/) → ○

　Android SDK の [Testing Support Library](http://developer.android.com/intl/ja/tools/testing-support-library/index.html) に含まれる [UIAutomator Viewer](https://www.youtube.com/watch?v=uA54T6R8nhs) のようなもの。これも Visual Studio の機能とは競合しないと思うので、継続されるでしょう。

## 価格

　みなさんが一番期待しているのは価格でしょう。今は BUSINESS EDITION(Android/iOS) で[年間20万円超](https://store.xamarin.com/)（個人向けなら月約5000円から）。

　Microsoft もここの売上をアテにしてるとは到底思えないので、恐らく何らかの改善があるのではないかと思います。
　無難なところでは 「MSDN Subscription に含まれる」でしょうか。

　完全無料化されて、 Visual Studio Community でも使用可能、になるととても嬉しいですね。

# まとめ

　2年前の投稿、[マカーの人が Xamarin について勘違いしていそうな５つのこと](http://qiita.com/amay077/items/2e86b44e5f274a34b2e9) で、以下のように書きました。

> 私は独立した企業である現在のポジションが Xamarin社にとってベストだと思っています。(中略) Evolve2014 には、Microsoft の他に IBM, Amazon, Google, Salesforce, Dropbox と言った、他ではちょっと見られないような豪華なスポンサー群になりました。これも Xamarin の中立な立ち位置がなせる技だと思います。

　「中立な立場の方が良いのでは？」という個人の意見は変わっていません、今度の Evolve にも [Apple の Steve Wozniak が参加する](https://blog.xamarin.com/join-apple-co-founder-steve-wozniak-at-xamarin-evolve-2016/) と話題になっているのですが、さすが Microsoft のイベントには来られないよなー、と思います。

　とはいえ、「いずれ・・・」と思っていたのも事実で、「ついにXデーが来たかー」、という感想です。
　マイクロソフトももはやガチガチのプロプライエタリというよりはかなりオープンな企業になっていますし、それには Xamarin（というか Mono）の活動も少なからず影響を与えていたと思います（[.NETがオープンソース化される](https://msdn.microsoft.com/ja-jp/library/dn878908(v=vs.110).aspx)とは、数年前誰が予想していたでしょうか）。
　上でまとめて来たように、マイクロソフトにとってメリットの多い買収なので、少なくとも飼い殺しのような事にはならないでしょう。

　個人的にも、「Xamarin が加わった新しい Microsoft」に期待して、Xamarin.Android を始めとした Xamarin 製品群を使い続けますし、Qiita を始め Tips の投稿もしていくつもりです。
　
(おまけ)

今回の買収劇のオチは、

**「MSを助ける製品の販売代理店として頑張って活動していたら、その製品がMSに買収されてなくなっちゃった」**

という[某さん](https://twitter.com/ytabuchi/status/702634391957217280)でしょうかw
　