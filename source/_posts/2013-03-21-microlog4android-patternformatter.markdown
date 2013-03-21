---
layout: post
title: "microlog4android で PatternFormatter を使う"
date: 2010-12-26 6:03
comments: true
categories: [Android, Java]
---
前回 から連続ポストですが、 [microlog4android](http://code.google.com/p/microlog4android/) の Formatter の中の一つ PatternFormatter について調べたのでメモ。
<!--more-->

[Log4j] にも PatternFormatter はあるので同じかなーと思いつつ、少し違うようです。

microlog4android の[ソースコード](https://github.com/johanlkarlsson/microlog4android) から、使える項目の説明を抜粋したのが以下です。

* %i : the client id
* %c : prints the name of the Logger
* %d : prints the date (absolute time)
* %m : prints the logged message
* %P : prints the priority, i.e. Level of the message.
* %r : prints the relative time of the logging. (The first logging is done at time 0.)
* %t : prints the thread name.
* %T : prints the Throwable object.
* %% : prints the ‘%’ sign.

よく使うのは %d, %m, %P, %T って所でしょうか。

中でも %d 時刻の出力について詳しく見ていきます。

%d を普通に使うと以下のような出力になります。ex:“%d [%P] %m %T”

> 13:07:27,683 [INFO] information

%d には {} でフォーマットが指定できます。使用できるのは、ABSOLUTE / DATE / ISO8601 の３種類。これも ソースコード から抜粋。何も指定しないと（つまり上記は）ABSOLUTE という事です。

次に DATE を指定した場合です。ex:“%d{DATE} [%P] %m %T”

> 26 DEC 2010 13:34:20,120 [INFO] information

最後に ISO8601 の場合です。ex:“%d{ISO8601} [%P] %m %T”

2010-12-26 13:36:40,684 [INFO] information

日本人的に一番みやすいのは ISO8601 ですかね。文字列でソートしやすいし。

[Log4j] では、 %d{yyyy-MM-dd} って感じで直接書式を指定できるっぽいですが microlog4android ではサポートされていないようです。実装されてる様子もありませんでした。

(余談ですが、microlog4android の中の実装では String.format とか SimpleDateFormat とか使われていませんでした。思うに、コイツらメチャ遅いからだと思います。)

最後に使い方はこんな感じです。

### Using PatternFormatter example

```java
// initialize logger
PatternFormatter formatter = new PatternFormatter();
formatter.setPattern("%d{ISO8601} [%P] %m %T");
logger.setLevel(Level.INFO);
// write to LogCat
LogCatAppender logCatAppender = new LogCatAppender();
logCatAppender.setFormatter(formatter);
logger.addAppender(logCatAppender);
// write to text file of SD-card.(need WRITE_EXTERNAL_STORAGE permission)
FileAppender fileAppender = new FileAppender();
fileAppender.setAppend(true);
fileAppender.setFileName("microlog4android.log");
fileAppender.setFormatter(formatter);
logger.addAppender(fileAppender);
```

前回のソースコードと照らし合わせてみてください。

#### 2010.12.27 追記
うぉ！時刻が UTC(GMT?) で出力されますね。まあいいか。