---
layout: post
title: "Xamarin.Forms の画面(Page)のライフサイクルイベントについて"
date: 2016-06-8 00:00:00 +0900
comments: true
categories: [Xamarin, Android, Xamarin.Forms, iOS]
---

ちょっと誤解してた＆情報がなかったのでまとめてみました。
<!--more-->

## モバイルアプリでの「画面」の基本的なライフサイクルイベント

まあ Android と iOS についてですが。

画面が表示される時のイベント（コールバック）は、簡潔には以下のようになります。

|順番|イベント|iOS(UIViewController)|Android(Activity)|
|---|---|---|---|
|1|画面がロードされる(た)時|viewDidLoad|onCreate|
|2|画面が表示される(た)時|viewDidAppear|onResume|
|3|画面が非表示になる(った)時|viewDidDisappear|onPause|
|4|画面がアンロードされる(た)時|viewDidUnload|onDestroy|

厳密にはもっと細かく、〜される前と後が iOS と Android で微妙に異なるのでだいたいこんな感じという程度と思って下さい。

もう少し細かいイベントは以前調べた以下を参考にしてみてください。

* [iOS と Android で画面表示時のコールバックを比較する - Qiita](http://qiita.com/amay077/items/52a0b0da97fe455abc08)

## Xamarin.Forms での画面のライフサイクルイベント

Xamarin.Forms では、上表のライフサイクルイベントは、アプリケーション
(Application) と、画面(Page) のイベントに分かれています。 

表に、Xamarin.Forms を追加してみました。

|順番|イベント|iOS(UIViewController)|Android(Activity)|Xamarin.Forms|
|---|---|---|---|---|
|1|画面がロードされる(た)時|viewDidLoad|onCreate|[Page.OnAppearing](https://developer.xamarin.com/api/member/Xamarin.Forms.Page.OnAppearing()/)|
|2|画面が表示される(た)時|viewDidAppear|onResume|[Application.OnResume](https://developer.xamarin.com/api/member/Xamarin.Forms.Application.OnResume()/) **※要注意** |
|3|画面が非表示になる(った)時|viewDidDisappear|onPause|[Application.OnSleep](https://developer.xamarin.com/api/member/Xamarin.Forms.Application.OnSleep()/)
|4|画面がアンロードされる(た)時|viewDidUnload|onDestroy|[Page.OnDisappearing](https://developer.xamarin.com/api/member/Xamarin.Forms.Page.OnDisappearing()/)|

画面のロード時（``viewDidLoad/onCreate``）に相当するのは、Xamarin.Forms では ``OnAppearing``、逆にアンロード時は ``OnDisAppearing`` です（名称が ``viewDidAppear`` に似てるので、画面の表示時かと勘違いしてました）。

画面の表示／非表示時のイベントは、画面でなく Application クラスの ``OnResume``, ``OnSleep`` で提供されます。

要注意なのは ``OnResume`` です。これ、画面が表示される **初回はイベントが発生しません**。
一度、アプリを背面に退避し、再度前面に持ってきたときに初めて ``OnResume`` が呼び出されます。iOS や Android の ``viewDidAppear / OnResume`` と同じだと思ってつかうとハマります。

基本的には、 ``OnAppearing`` でリソースの確保を、 ``OnDisAppearing`` で解放をすればよさそうです。

GPS など、電池消費の激しいリソースを使う場合は、アプリが背面へ隠れたら直ちにそのリソースを解放した方がよいです。その場合は ``OnResume-OnSleep`` を使いますが、前述の通り、画面初回表示時は ``OnResume`` が走らないので、少し工夫が必要です。

## 画面が回転された時

端末を横向きに回転すると、Android では Activity が破棄されて再度生成される事が知られています。

Xamarin.Forms では Android でもそのような事はなく、 ``Page.OnSizeAllocated`` が呼び出されるだけです(iOS も当然同じ)。

## 参考

* [Working with the App Lifecycle - Xamarin](https://developer.xamarin.com/guides/xamarin-forms/working-with/app-lifecycle/)
