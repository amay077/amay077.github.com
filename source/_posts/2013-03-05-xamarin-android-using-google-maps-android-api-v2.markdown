---
layout: post
title: "Xamarin.Android で Google Maps Android API v2 を使う"
date: 2013-03-05 23:10
comments: true
categories: [Xamarin, Android, C#, GoogleMapsAPI]
---
だんだん自力でサンプルプログラム書くのに疲れてきたので、MonoDroid のサンプル集に頼ることにします。

* [xamarin / monodroid-samples](https://github.com/xamarin/monodroid-samples)

<!--more-->

すごく豊富なので、これひと通り動かせば大抵のことはできるようになるんじゃないか。。。

というわけで、この中の [MapsAndLocationDemo_v2](https://github.com/xamarin/monodroid-samples/tree/master/MapsAndLocationDemo_v2) を動かしてみます。Readme.md に沿って。

## ソースを取得する
まず monodroid-samples を Clone しときます。

> $git clone git://github.com/xamarin/monodroid-samples.git

## クライアントライブラリをビルドする(Building the Client Library)
ここでいうクライアントライブラリとは、Android SDK に含まれる **Google Play services** の事です。
この準備は本家Android開発で Google Maps Android API v2 を使うときに必要なので、これが何者かは、

* [throw Life - Google Maps Android API v2を使ってみた](http://www.adamrocker.com/blog/334/google-maps-android-api-v2.html)

などを参考に。
SDK で Google Play services が取得できていれば、

``{※あなたのAndroidSDKディレクトリ}/extras/google/google_play_services/libproject/google-play-services_lib`` ができているはず。

※Xamarin.Android で使われる Android SDK は、Xamarin Studio のPreferences → SDK Locations → Android で確認できます。

!["sdklocations"](https://dl.dropbox.com/u/264530/qiita/xamarin_android_sdk_locations.png)


``google-play-services_lib`` へ移動して、Ant 用のビルドファイルを生成します。

> $cd google-play-services_lib
> $Android update project --path .
> $ant debug

Readme.md では ``android project update -p .`` って書いてあるけど上のコマンドじゃないとうまくいかないんですけど？
あと ``{あなたのAndroidSDKディレクトリ}/tools`` にパス通しておかないと android コマンド使えないです。ant も(ry

ant の実行に成功すると /bin の中に classes.jar とかができてるはずです。

次に、Xamarin Studio で ``MapsAndLocationDemo.sln`` を開いて、GooglePlayService プロジェクトを展開します。
すると、`project.properties`` ファイルが赤くなってる(ちゃんと指定した位置に google-play-services_lib をコピってれば赤くなってないかも)ので、これを一度削除して、追加しなおします。リンクファイルとして、ね。

一連の画像貼っときます。

!["red"](https://dl.dropbox.com/u/264530/qiita/xamarin_studio_delete_file.png)
!["red"](https://dl.dropbox.com/u/264530/qiita/xamarin_studio_add_exist_file.png)
!["red"](https://dl.dropbox.com/u/264530/qiita/xamarin_studio_add_a_link.png)


## Google Maps v2 API Key

Google Maps v2 API Key の取得も本家と同じですが、
ディレクトリが、``/Users/[USERNAME]/.local/share/Xamarin/Mono for Android/debug.keystore`` なのでそこだけ注意。(←Mac の場合。Win は Readme.md 見て)

該当のディレクトリへ移動して、以下のコマンドで表示されるキーの内、SHA-1 をコピーします。

>$ keytool -V -list -keystore debug.keystore -alias androiddebugkey -storepass android -keypass android

あとは、

* [throw Life - Google Maps Android API v2を使ってみた](http://www.adamrocker.com/blog/334/google-maps-android-api-v2.html)

を参考に、Google API Console から、API key を取得します。
API Key を取得する際に必要なパッケージ名、取得した API Key を貼り付ける場所は、いずれも MapsAndLocationDemo プロジェクトの AndroidManifest.xml にあるので、先に開いておくとよいでしょう。

!["apikey"](https://dl.dropbox.com/u/264530/qiita/xamarin_studio_android_manifest.png)

API Key を取得したら、``com.google.android.maps.v2.API_KEY`` のところに貼り付けて準備完了です。

## A Note About Google Play Services

Google Play Services が端末にインストールされてないと動かんよ、つまりエミュレータでは動かんよ。Google Play Services のインストールの仕方など、私の知ったことではない。だそうです。

## 動かしてみる

「実機で」動かしてみます。

!["play"](https://dl.dropbox.com/u/264530/qiita/xamarin_studio_android_google_maps_api_v2.png)

動いてます。(上は Show Map with Overlays を動かしたところ)

今回はここまで。コードの深追いは日を改めて。