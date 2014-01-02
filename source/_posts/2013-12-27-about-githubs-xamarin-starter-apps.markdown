---
layout: post
title: "GitHub's Xamarin starter apps  について"
date: 2013-12-22 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#]
---
Github社は、割と Xamarin に熱心で、[Xamarin Evolve というイベントでセッション](http://xamarin.com/evolve/2013#session-zm59b5yptf)を行ったり、その時の資料が日本の速報系サイトで 「[Ruby を捨てて C# と MVVM に完全移行](http://blog.livedoor.jp/itsoku/archives/33671593.html)」なんてヒドい扱い受けちゃったりしてます。
<!--more-->
でその Github 社が Xamarin でアプリ開発するならこれ使うといいよ、的なアプリケーションのひな形？を公開しています。

* [GitHub's Xamarin starter apps](http://log.paulbetts.org/open-source-githubs-xamarin-starter-apps/)

Github社は、自前の MVVM フレームワーク「ReactiveUI」、非同期KVS の 「Akavache」を作ってますので、それを利用したものになっています。

* [ReactiveUI](http://www.reactiveui.net/)
* [Akavache](https://github.com/akavache/Akavache)

Starter Apps を Xamarin Studio で開くとこんな感じです。

![](https://dl.dropboxusercontent.com/u/264530/qiita/xamarin_startup_apps_by_github_01.png)

アプリケーション自体は MVVM で作られていて、前述の RectiveUI によって、View 以外は極力プラットフォームに依存しないように作ることができます。

Starter-Core-Android と Starter-Core-iOS が、「View以外」の部分に相当します。(ここ PCL化 できれば１プロジェクトで済みそうですが)

Starter-Android と Starter-iOS は、各プラットフォームの View に相当します。

かくいう私もまだソースをじっくり読んでないのですが、クロスプラットフォームで開発する時の教材になるかなーと思います。