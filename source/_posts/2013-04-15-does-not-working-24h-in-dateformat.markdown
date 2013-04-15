---
layout: post
title: "DateFormat.format では HH:mm が使えない"
date: 2013-04-15 19:34
comments: true
categories: [Android]
---
Android には ``android.text.format.DateFormat`` というユーティリティクラスがあるのですが、これの ``format`` メソッド、時刻の24時間形式に対応してません。
<!--more-->
```java DateTimeFormatTest.java
long t = System.currentTimeMillis();
Log.d("By DateFormat", DateFormat.format("HH:mm", t).toString());
Log.d("By SimpleDateFormat", new SimpleDateFormat("HH:mm").format(new Date(t)));
```

```
04-09 22:42:14.435: D/By DateFormat(2860): HH:42 ←あ〜あ
04-09 22:42:14.435: D/By SimpleDateFormat(2860): 22:42
```

Staclkoverflow とかにも「代わりに SimpleDateToimeFormat を使えば」と書いてあるわけですが、いやいやそれじゃユーティリティクラスの役割果たしてないでしょ？と思うところであります。

自分の中では、時刻は24h表示がデフォルトなので、「DateFormat クラス使えない子」というレッテルを貼ってしまいました。

## 参考

*[java - 24 hour clock not working in dateformat android - Stack Overflow](http://stackoverflow.com/questions/5755073/24-hour-clock-not-working-in-dateformat-android)