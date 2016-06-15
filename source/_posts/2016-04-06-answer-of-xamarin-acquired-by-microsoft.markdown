---
layout: post
title: "Xamarin が Microsoft に買収された結果"
date: 2016-04-01 23:59:59 +0900
comments: true
categories: [Xamarin, C#, Android, iOS]
---
[Xamarin が Microsoft に買収されたので、今後を勝手に予想](http://qiita.com/amay077/items/4aa25db9509216cf5bf0) の答え合わせなんですが…
<!--more-->
* [Xamarin for Everyone | Xamarin Blog](https://blog.xamarin.com/xamarin-for-all/)
* [【速報】Xamarin のこれからについて！ - Xamarin 日本語情報](http://ytabuchi.hatenablog.com/entry/ms-xamarin)

**Xamarin は無料ですべての Visual Studio に同梱されることになりました！**

**Mac では Xamarin Studio が無料で使えるようになりました！！** [※注](http://qiita.com/amay077/items/6e5c40abe0c21fc79e6a#%E8%BF%BD%E8%A8%98-to-%E4%BC%81%E6%A5%AD%E3%81%AE%E4%BA%BA%E7%84%A1%E6%96%99%E3%81%AB%E3%81%AA%E3%82%8B%E3%81%A8%E8%A8%80%E3%81%A3%E3%81%9F%E3%81%AA%E3%81%82%E3%82%8C%E3%81%AF-visual-studio-pro-%E4%BB%A5%E4%B8%8A%E3%82%92%E6%8C%81%E3%81%A3%E3%81%A6%E3%82%8B%E4%BA%BA%E3%81%AE%E3%81%BF%E3%81%A0)

というか、 **Xamarin のコアライブラリがオープンソースになりました！！！**

今日も仕事が手につきませんね！


## [Xamarin.Android, Xamarin.iOS](https://xamarin.com/platform) → ◎◎◎

期待以上でしたね。
プロダクトとしては無償になります。
すべての機能が制約なしに使えます。
ソースコードが MIT Lisence なオープンソースになります。

## [Xamarin.Mac](https://xamarin.com/platform#desktop) → ◎

> OSS として公開、のような可能性があるなら嬉しいかも。

これ当たりましたね。Xamarin.Mac の人もこれで一安心。


## [Xamarin.Forms](https://xamarin.com/forms) → ◎◎◎

これも上2つど同様にオープンソースに。
正直しばらくはプロプラエタリでいくかなーと思ってたので、完全に期待以上でした。
UIデザイナーは・・・Evolve？

## [Xamarin Studio](https://xamarin.com/studio) → ◎

これも無償化。ここのソースコードはOSSなのかな？ → OSSにはならないようです（もちろん元々OSSであるMonoDevelop以外のXamarin固有のプラグインのこと） - [How do I share code across platforms with Xamarin? and other FAQs - Xamarin](https://www.xamarin.com/faq#xpq7)
とりあえずMacでの開発者には嬉しい。

そして、ありがとう、[さよなら Xamarin Studio for Windows](https://www.xamarin.com/faq#xpq6)

## [Xamarin Components](https://components.xamarin.com/) → ？

まあ、消える流れですよね。。

## [Xamarin Test cloud](https://xamarin.com/test-cloud) → ◎◎

Visual Studio Team Services に同梱されるとのことです。
[その価格](https://www.visualstudio.com/ja-jp/products/visual-studio-team-services-pricing-vs.aspx) を見ると、今までよりグッと使いやすくなりました。

## [Xamarin Insights](https://xamarin.com/insights) → ◎◎

> 独立させて残す意味はあまりないので、 Application Insights に統合されていくのかなあ、という感じです。

これハズレましたね。

[HockeyApp](http://hockeyapp.net/features/) というサービスに統合されるとのことです。

HockeyApp って知らなかったのですが、ログ収集・解析の他に、DeployGate のような配布機能も持っているみたいですね。これは嬉しい。

## [RoboVM](https://robovm.com/) → ？

Build2016 では予想通り全く触れられませんでした。さて未来は？

## その他

### [Xamarin Android Player(Preview)](https://developer.xamarin.com/guides/android/getting_started/installation/android-player/) → ？

Build2016 では Windows上で動く iOS Simulator のデモを行っていました。

Miguel de Icaza曰く

> "Have touch and no need to turn to your Mac"

だそうですよ。


### [Xamarin Profiler(Preview)](https://xamarin.com/profiler) → ？

これは特に情報ありませんでした。

### [Xamarin Inspector(Early Preview)](https://developer.xamarin.com/guides/cross-platform/inspector/) → ？

Windows の Android エミュレータ上でアプリが動いているところで、コードを変更すると、 **即座にアプリに変更が適用される** という謎のデモを行っていました。

Xamarin（もとい Microsoft ）の中の人曰く、

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">これだよこれがインスタントプログラミングだよ!</p>&mdash; Atsushi Eno (@atsushieno) <a href="https://twitter.com/atsushieno/status/715566438203809792">2016年3月31日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

とのことです。（Android Studio さん聴いてる？）

Keynote Day 2 の動画あったのでデモ見られます→ [Microsoft Build 2016 | Keynote Day 2 (HD) - YouTube](https://www.youtube.com/watch?v=WC7ijoFzjEg&feature=youtu.be&t=16m) （このデモの後ほどなくして "making Xamarin available at no extra charge." →大歓声、ですね）
# まとめ

ということで、総じて言えば、皆さんの期待にすべて応える感じで、Xamarin のパワーを活かして開発者を増やしていきたいという意思が表れていましたね。

本当に想像以上でした。
Indie ライセンス買ったばかりだけど、そのお金返せとは言いません、ありがとう Xamarin！！

## 【追記】 to 企業の人、無料になると言ったな、あれは Visual Studio Pro 以上を持ってる人のみだ

Mac しか使ってない企業が自社のアプリを Xamarin を使って開発・配布する場合、 Xamarin Studio を使うことになります。

[Store - Xamarin](https://store.xamarin.com/) の Xamarin Studio にある Small teams をクリックすると [MICROSOFT VISUAL STUDIO COMMUNITY 2015](https://www.visualstudio.com/support/legal/mt171547) が表示され、以下のような記述があります。

> 1. インストールおよび使用に関する権利。
> 
> b. 組織ライセンス。お客様が組織である場合、お客様のユーザーは以下の条件で本ソフトウェアを使用することができます。
> 
> * お客様がエンタープライズである場合、お客様の従業員および契約社員は本ソフトウェアを使用して、お客様のアプリケーションを開発またはテストすることはできません。ただし、上記で許可されているオープンソースおよび教育目的の場合を除きます。「エンタープライズ」とは、合計で (a) 250 台を超えるコンピューターがある、もしくは 250 人を超えるユーザーがいる、 または (b) 年間収益が 100 万米ドル (もしくは他の通貨での相当額) を超える、組織およびその関連会社のことです。「関連会社」とは、組織を (過半数所有により) 支配している法人、組織が支配している法人、または組織と共通の支配下にある法人を意味します。

~~組織（企業）での利用で、250人を超えるユーザー(=配布スマホ台数ということになるでしょう)が居る場合は、使用できない、と読み取れます。~~
[コメント](http://qiita.com/amay077/items/6e5c40abe0c21fc79e6a#comment-2297416c6d83b3593425)で教えていただきました。日本語の [Visual Studio Community のページ](https://www.microsoft.com/ja-jp/dev/products/community.aspx) には、ユーザー数に関する記述はないので、この点（配布スマホ台数）に関しては気にしなくてもよさそうです。

また、 Xamarin の FAQ - [How do I share code across platforms with Xamarin? and other FAQs - Xamarin](https://www.xamarin.com/faq#xpq8) には、以下の記述があります。

> Xamarin Studio will follow the Visual Studio pricing rules. There is Xamarin Studio Community Edition available for download on the Mac. You’ll need to be a Visual Studio Enterprise subscriber to unlock Visual Studio Enterprise features in Xamarin Studio.

Xamarin Studio は、Visual Studio の価格体系に従う、とのことなので、ライセンス条項も同じだと解釈すれば、 Visual Studio Profesional 以上のライセンスがあれば、 Mac + Xamarin Studio で企業のアプリを開発・配布して問題ないと言えます。
~~Macオンリー企業には、ツールとしてでなくライセンスとしての Visual Studio Pro 以上が必要ということなるのでしょうか。~~

* [【お知らせ】Xamarin ライセンスの移管について - Xamarin 日本語情報](http://ytabuchi.hatenablog.com/entry/2016/04/21/123000)

によると、 企業向け開発する場合、 Windows ＋ Visual Studio は Visual Studio Pro単品購入で可能、Mac + Xamarin Studio には Visual Studio Pro **MSDN Subscription** が必要とのことです。
