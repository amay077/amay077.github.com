---
layout: post
title: "Xamarin.Android が使用する debug.keystore の在処"
date: 2014-08-28 17:42:04 +0900
comments: true
categories: [Xamarin, Android]
---
Google Maps などを使用する時、開発中ならば ``debug.keystore`` のフィンガープリント(SHA1)を取得する必要がありますが、 debug.keystore がどこにあるのか分からなくてハマった。
<!--more-->
## 結論

から書いておくと

* [Obtaining a Google Maps API Key | Xamarin](http://developer.xamarin.com/guides/android/platform_features/maps_and_location/maps/obtaining_a_google_maps_api_key/)

に書いてある通りで、

* Windows - ``C:¥Users¥[USERNAME]¥AppData¥Local¥Xamarin¥Mono for Android¥debug.keystore``
* OSX - ``/Users/[USERNAME]/.local/share/Xamarin/Mono for Android/debug.keystore``

がそれぞれ使われる。

## なぜハマったか？

### Eclipse と同じだと思ってた

Java での Android 開発時に設定したディレクトリを使ってくれると思い込んでた。けどよく考えればあれは Android SDK ではなく Eclipse 固有の設定だったのよね。

### ドキュメントが古いと思ってた

上記で紹介した「Obtaining a Google Maps…」の記事が古いと思ってた。なぜならディレクトリ名に ``Mono for Android`` が含まれていて、これは Xamarin.Android の旧製品名だったから。

### Spotlight検索でヒットしなかった

Mac を使っているのだけど、Spotlight検索（所謂PC内検索）で ``debug.keystore`` がヒットしたのがいつもJava-Android開発で使ってる１つだけだったので、.local の中にあるとは気付けず。。。隠しディレクトリだからヒットしないよね。

こんなことでハマるのは自分くらいだろうけど、メモしときます。。