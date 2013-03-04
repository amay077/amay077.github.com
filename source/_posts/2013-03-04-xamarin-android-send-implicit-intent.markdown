---
layout: post
title: "Xamarin.Android で暗黙的Intent を発行する"
date: 2013-03-04 23:07
comments: true
categories: [Xamarin, Android, C#]
---
まあ特に特別なことはないんですが。。。

<!-- more -->

## 電話をかける

```c#
var intent = new Intent(Intent.ActionCall, 
    Android.Net.Uri.Parse("tel:1234567890"));
context.StartActivity(intent);
```
※ 電話をかけるには ``CALL_PHONE`` 権限が必要です。権限の追加の仕方は[こちら](http://amay077.github.com/blog/2013/03/02/xamarin-android-permission/)。

## 地図を呼び出す

```c#
var uri = String.Format("geo:{0},{1}?q={0},{1}({2})", 
    35.710211, 139.810874,
    "東京スカイツリー");

var intent = new Intent(Intent.ActionView, Android.Net.Uri.Parse(uri));
context.StartActivity(intent);
```

``Android.Net.Uri`` クラスは ``System.Uri`` クラスで代替できると嬉しかったかも。ちょっとだけ。いやダメか。