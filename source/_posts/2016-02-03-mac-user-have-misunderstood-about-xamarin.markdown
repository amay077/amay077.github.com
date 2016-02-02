---
layout: post
title: "マカーの人が Xamarin について勘違いしていそうな５つのこと"
date: 2014-12-02 00:00:00 +0900
comments: true
categories: [Xamarin, iOS, Android]
---

　今年も始まりました [Xamarin Advent Calendar 2014](http://qiita.com/advent-calendar/2014/xamarin) 。
　思えば[昨年の Advent Calendar](http://qiita.com/advent-calendar/2013/xamarin) は、5名の方に参加していただいたものの「ほとんど俺」みたいな感じでしたが、今年はたくさん方に登録してもらえてとても嬉しいです。この1年の Xamarin の躍進ぶりを象徴していると思います。
<!--more-->

　そんな Xamarin ですが、Microsoft とのパートナーシップが強力なおかげで、Windows系の開発者には広く知られて（そういう戦略なのは分かります）いますが、普段 Mac で iOS/Android アプリを開発してますみたいな人にはあまりリーチできていないかなあと思います。

　そこで初日の今日は、マカーの人が、勘違い・思い込んでいそうなことをいくつか払拭してみたいと思います。

## Q1. Xamarin を使うには、Visual Studio が必要なんでしょ？

A1: **必要ありません。** 「Xamarin Studio」という専用の統合開発環境で開発できます。私は Mac + Xamarin Studio で開発していますが、まったく問題を感じていません。
　また、iOSアプリのView部分は、Xcodeと同じ ``.storyboard`` ファイルを使用しますが、Xcode を使う必要もありません。Interface Builder と同じ（か部分的にはそれ以上）の機能を持つ [UIデザイナー](http://developer.xamarin.com/guides/ios/user_interface/designer/)が、Xamarin Studio には搭載されています。

## Q2. Xamarin を使うには、Windows が必要なんでしょ？

A2: **必要ありません。** Mac のみで完結します。むしろ Windows だけでは iOSアプリのビルドができないので、[Mac にリモート接続](http://developer.xamarin.com/guides/ios/getting_started/installation/windows/introduction_to_xamarin_ios_for_visual_studio/)する必要があり、これがしばしばトラブルになります。（主にデモでｗ
　Microsoft がアピールするとどうしても Windows+Visual Studioの説明になってしまいますが、それはまやかしです（言い切った！

## Q3. Xamarin社って、Microsoft の子分みたいなもんでしょ？

A3: Xamarin社は独立した企業であり、Microsoftとは対等な立場です（と私は思っています）。「Microsoft に買収されればいいのに」という声をよく聞きますが、私は独立した企業である現在のポジションが Xamarin社にとってベストだと思っています。Microsoftにとってはモバイル開発者にリーチする重要なピースであり、Xamarin社としても他にないマーケットです。
 また、今年の Xamarin の大イベント [Evolve2014](https://evolve.xamarin.com/) には、Microsoft の他に IBM, Amazon, Google, Salesforce, Dropbox と言った、他ではちょっと見られないような豪華なスポンサー群になりました。これも Xamarin の中立な立ち位置がなせる技だと思います。
あ、最近の [.NETのオープン化](http://www.publickey1.jp/blog/14/jitnet_core_rutimenet_framework.html) の流れは、Xamarin の CTO であり Monoプロジェクトの生みの親であるスーパーハッカー、[ミゲル・デ・イカザ](https://twitter.com/migueldeicaza)氏が少なからず関係していると思っています。

## Q4. C# 覚えるのしんどい

A4. **あなたはあの Objective-C を覚えたのでしょう？**

## Q5. Swift の方が C# よりイケてるじゃん？

A5. 後発である Swift がイケてるのは誰もが認めるところでしょう（かつて Java に対する C# がそうであったように）「Swift は関数型言語だ」という意見には、Xamarin は F# を提案します。[Xamarinには F# の MVP（勝手に”数学ガール”だと思っている）](http://blog.xamarin.com/introduction-to-f-with-xamarin/)も居ます(←訂正:Xamarinの人じゃなかったです)し、日本でも [F#+Xamarin でアプリ開発されている型](http://www.slideshare.net/kusokuzeshiki/xamarinmvvm-crossf)も居らっしゃいます。

# まとめ

　ちょっと宗教論争っぽくなりかけたので、ここまでにしておきます。強く主張したいのは、モバイルアプリ開発者なら iOS だけ、Android だけ知っていても良いアプリは作れないでしょう。両方のプラットフォーム、開発言語、哲学を理解する必要があります。 **Xamarin だから Swift を覚えなくていいという事はありません。** 
　
　でも、同じ（少なくとも同じような機能をもった）アプリの同じロジックを、異なる言語でそれぞれ書いて、その後数年保守し続ける現状は、本当に最適なのでしょうか？同じコード、あるいは同じバイナリが iOS/Android で動作すれば、保守費用は半分です（SIer みたいな言い方だｗ）。

　「共通にできる選択肢、あるいはプラットフォームの文化にあわせて別々にできる選択肢」を自然な形で提供するのが Xamarin、 Java も Swift も C# も覚えて C# で D.R.Y するのが Xamarin です。

　最後に宣伝ですが、 **Build INSIDER** というWebサイトで「Xamarin逆引きTips」という連載をしています。 

* [Xamarin逆引きTips - Build Insider](http://www.buildinsider.net/mobile/xamarintips)

　これは、.NET Framework は今まであまり使った事がない iOS/Android アプリ開発者をターゲットにしていて、説明もほぼ全てが Mac+Xamarin Studio を使って書いています。興味持ったら読んでもらえると嬉しいです。

　本日まったく登場しなかった Visual Studio や Windows Phone などの話は、明日以降登場すると思いますので、お楽しみに！それでは初日はこの辺で。
