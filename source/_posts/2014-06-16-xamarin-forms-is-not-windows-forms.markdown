---
layout: post
title: "Xamarin.Forms は Windows.Forms じゃないよ"
date: 2014-06-08 15:28
comments: true
categories: [Xamarin]
---

* [iOS／Androidの画面レイアウトを共通化するには？（Xamarin.Forms） - Build Insider](http://www.buildinsider.net/mobile/xamarintips/0005)

に書ききれなかったことなど。
<!--more-->

## 名前どうよ？

**Xamarin.Forms** って名前、どうしても Windows.Forms を連想するけど全然別ものだから。とはいえ、あんまり WPF の経験がないので、Windows.Forms にそれほどイヤな印象がない。VB6時代なんて、画面に Fill するのすらゴリゴリ実装が必要だったし。

## 前身は MonoTouch.Dialog じゃないかと思います

MonoTouch.Dialog というのは、Xamarin の CTO でありスーパーハッカーのミゲル氏が開発した、簡単なコードで iOS の UI を実装できるライブラリ（「Dialog」だけど、「ダイアログボックス」とは関係ない。 ）

* [migueldeicaza/MonoTouch.Dialog](https://github.com/migueldeicaza/MonoTouch.Dialog)

たぶんだけど、Xamarin.Forms はこれの思想がベースにあって、他のプラットフォームやXAML対応が行われたんじゃないかと勝手に思ってます。（いつか紹介しようと思ってたけどこれでお役御免になってしまったかも）

## Android の UIフレームワークに似てるような…

[StackLayout](http://developer.xamarin.com/guides/cross-platform/xamarin-forms/controls/layouts/) は LinearLayout相当、RelativeLayout も、同名クラスがあるし、割と Android の UI の考え方をベースにしてるのかなと予想。Windows Phone はあまり経験がないので知らない。。。

## Hoge.iOS というプロジェクト名がキライ

プラットフォームの識別子をプロジェクト名につけるのが良いんだけど、 ``HelloForms.iOS`` とか「iOS」とつけるのが嫌い。名前空間名とかが小文字で始まってしまうのが気持ち悪くて。
MvvmCross などでは、iOS は ``XXX.Touch`` 、Android は ``XXX.Droid`` としていて、こっちの方が好み。

## XAML はデザイナーが無いとツラい

WPFじゃなくてもXAMLって言っていいんだ、というのを初めて知ったｗ
Xamarin Studio のXMLデザイナーは補完も効かないしツラい。コードでUI作るのにも限界があるので、今年の [Evolve](https://evolve.xamarin.com/) あたりで「Xamarin.Forms のUIデザイナー発表ばばーん！！」を期待してます。

## タブの位置が上とか下とかどうでもいい

「UIはプラットフォームの流儀に合わせるべき」という説明に必ずと言っていいほど登場するのがタブバーの位置が Androidだと上で、iOSだと下、ってやつ。
個人的にはどうでもいい。タブの切り替えをよく使うなら指の届きやすい画面下部に配置すべきだし、同じアプリならiOS版とAndroid版でUIを変えるべきでないと思っている。（かと言って、iOS版のUIをそっくりそのままAndroidに移植すればいいという話でもない）。 **プロダクトのUXは、プラットフォームのUXより優先されるべき** 、と思ってる。

## 上司への提案には好材料かと

* 私「iOS/Android 両方開発するなら Xamarin 使いましょう！」
* 上「そうか、それは１ソースで両対応できるのか？」
* 私「いえ、画面はプラットフォーム毎に書きます、カメラとかも別です。共通化できるのはコアロジックです。」
* 上「なんだその程度か」
* 私「…」

というよくある話には、「画面が１ソースで共通にできる Xamarin.Forms」は、説得材料としては、とりわけ対PhoneGapには効果高いんじゃないかと思います。毒りんごかどうかはともかく。。。
