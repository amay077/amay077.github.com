---
layout: post
title: "DateFormat.format では HH:mm ではなく kk:mm を使う"
date: 2013-04-15 19:34
comments: true
categories: [Android, Java]
---
Android には ``android.text.format.DateFormat`` というユーティリティクラスがあるのですが、これの ``format`` メソッド、時刻の24時間形式に対応してません。
<!--more-->

```java DateTimeFormatTest.java
long t = System.currentTimeMillis();
Log.d("By DateFormat", DateFormat.format("HH:mm", t));
Log.d("By SimpleDateFormat", new SimpleDateFormat("HH:mm").format(new Date(t)));
```

```
04-09 22:42:14.435: D/By DateFormat(2860): HH:42 ←あ〜あ
04-09 22:42:14.435: D/By SimpleDateFormat(2860): 22:42
```

Staclkoverflow とかにも「代わりに SimpleDateToimeFormat を使えば」と書いてあるわけですが、いやいやそれじゃユーティリティクラスの役割果たしてないでしょ？と思うところであります。

<strike>自分の中では、時刻は24h表示がデフォルトなので、「DateFormat クラス使えない子」というレッテルを貼ってしまいました。</strike>

## 追記 2013.7.24
「HH でなく kk が使えるよ」とコメントで教えて頂きました。

```java 
DateFormat.format("kk:mm", t);
new SimpleDateFormat("kk:mm").format(new Date(t));
```

どうやら [SimpleDateFormat でも使える](http://stackoverflow.com/questions/8907509/how-to-set-24-hours-format-for-date-on-java)ようなので、「Java で 24h表記の場合は kk」って覚えておけばよさそうです。(k って何の略なの…)

## 参考
* [java - 24 hour clock not working in dateformat android - Stack Overflow](http://stackoverflow.com/questions/5755073/24-hour-clock-not-working-in-dateformat-android)

よく見たら、ここの Top Vote にも 「kk 使え」って書いてあったorz