---
layout: post
title: "Xamarin で Windows Azure モバイルサービスを使う(その2:認証編)"
date: 2013-12-18 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#, Azure, BaaS]
---
[前回](http://qiita.com/amay077/items/40bd5918284fd40d0edc) は、Microsoft の BaaS である Azureモバイルサービスをとりあえず使ってみる所まで紹介しました。

今回は、OAuth による認証、Twitter や Facebook でログインする機能について触れてみます。
<!--more-->
前回用意したサンプルの続きとして進めます。

認証を組み込むためのチュートリアルはこちら

* [Get started with authentication (Xamarin iOS) | Mobile Dev Center](http://www.windowsazure.com/en-us/develop/mobile/tutorials/get-started-with-users-xamarin-ios/)

英語版だからと謝罪してくれるのは Microsoft だけですね。

## 1. モバイルサービスに認証を設定する

> Register your app for authentication and configure Mobile Services

のところです。

まずは https://manage.windowsazure.com にアクセスし、モバイル サービス から、前回作った名前をクリックします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_2_01.png)

次に上部メニューからダッシュボードを選択します。表示されるページの左下の方にある「モバイルサービスURL」の値をコピーしておきます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_2_02.png)

次に、どのサービスの認証を利用するかを決めます（Microsoft, Facebook, Twitter, Google）。この説明では Twitter を使うことにします。

Twitter での操作方法は下を開きます。

* [モバイル サービスでの Twitter ログイン用のアプリケーションの登録](http://www.windowsazure.com/ja-jp/develop/mobile/how-to-guides/register-for-twitter-authentication/)

説明が充実してるので上記に従ってください。要は「コンシューマ キー」と「コンシューマー シークレット」が得られれば OK です。

ではマネージメントポータルの説明に戻ります。

> 4. Back in the Management Portal

です。

上部メニューの ID をクリック、その後、「twitter 設定」に先ほど取得した コンシューマ キー とコンシューマ シークレット を貼り付けて、一番下にある「保存」をクリックします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_2_03.png)

続いて、テーブルに権限を与えます

> Restrict permissions to authenticated users

のところ。

まず上部メニューの データ → TodoItem をクリックします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_2_04.png)

次に、アクセス許可 をクリックし、全ての項目で「認証されたユーザーのみ」を選択します。その後「保存」します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_2_05.png)

マネージメントポータル側の設定は以上です。
Twitter での認証設定を追加し、またテーブルには、認証されたユーザーでしかアクセスできない権限を設定しました。

結果、前回動かしたサンプルプログラムはこの設定を行う事で動作しなくなります。(401エラー)

次に Xamarin Studio でコードを変更していきます。

## 2. アプリケーションに認証を追加する

> Add authentication to the app

のところです。

1. 〜 7. まであります。流れは説明通りですが、クラス名や大文字小文字が微妙に違うので注意が必要です。(``TodoService`` は ``QSTodoService`` と読み替える必要があるなど)

これらを修正したソース２つ ``QSTodoService.cs`` と ``QSTodoListViewController.cs`` を gist に貼り付けました。

* [gist - QSTodoListViewController.cs and QSTodoService.cs](https://gist.github.com/amay077/7960424)

修正前との差分が見たい方は [Revisions](https://gist.github.com/amay077/7960424/revisions) で見られます。

あ、今回は Twitter なので、
``MobileServiceAuthenticationProvider.MicrosoftAccount`` を忘れずに 
``MobileServiceAuthenticationProvider.Twitter`` に置き換えてくださいね。

これでコードの修正は終わりです。

3. 動かしてみる

さて、iOS シミュレータ で動かしてみましょう。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_2_06.gif)

こんな感じで、Twitter による認証機能をアプリに組み込むことができました。

今日は以上です。次回は Push 通知を行ってみます。
