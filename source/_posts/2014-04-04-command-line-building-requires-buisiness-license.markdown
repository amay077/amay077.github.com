---
layout: post
title: "Xamarin でビルドを自動化するには Business 版以上が必要です"
date: 2014-04-04 15:31
comments: true
categories: [Xamarin]
---
Xamarin でも、リリース用アプリのビルド→署名→テストとか、自動化したいじゃないですか。
<!--more-->
Win/Visual Studio の場合は ``msbuild``、Mac/Xamarin Studio の場合は ``xbuild`` でそれが可能との情報を得て試してみたところ、、、


```sh
$ xbuild HogeAppAndroid.csproj 

XBuild Engine Version 3.2.6.0
Mono, Version 3.2.6.0
Copyright (C) Marek Sieradzki 2005-2008, Novell 2008-2011.

Build started 2014/04/04 13:13:59.
__________________________________________________
Project "...HogeAppAndroid/HogeAppAndroid/HogeAppAndroid.csproj" (default target(s)):
	Target _SetLatestTargetFrameworkVersion:
: error XA9008: Building from the command-line requires a Business License.
	Task "ResolveSdks" execution -- FAILED
	Done building target "_SetLatestTargetFrameworkVersion" in project "...HogeAppAndroid/HogeAppAndroid/HogeAppAndroid.csproj".-- FAILED
Done building project "...HogeAppAndroid/HogeAppAndroid/HogeAppAndroid.csproj".-- FAILED
```


**“Building from the command-line requires a Business License.”**

だそうです。

Starter Edition, Indie Edition では、コマンドラインからのビルドは許可されていないようです。

* [Store - Xamarin](https://store.xamarin.com/)

の比較表ではちょっと分からなかったな。。。

Win/Visual Studio な人は自動的に Business Edition 以上だから問題ないのですが、個人だから Business版買えないよ、Starter/Indie だけど (なんちゃって)CI とかやりたいよ、という人はあきらめるしかないみたいです。

…手動でやります。