---
layout: post
title: "Getting start of microlog4android"
date: 2010-12-26 3:13
comments: true
categories: [Android, Java]
---
[microlog4android](http://code.google.com/p/microlog4android/) は、Android 用のロガーライブラリ。

[Log4j](http://logging.apache.org/log4j/1.2/) のような使い方ができ、簡単に Android アプリケーションにログ機能を実装できます。
<!--more-->

もうちょっと詳しく言うと、Log4j を真似て作った [Microlog](http://microlog.sourceforge.net/site/) というモバイル用のロガーライブラリがあり、その中から Android に必要なものを切り出し+追加したのがこの microlog4android です。

元々は Log4j を Android で使おうと思って色々試していたのですがうまくいかなかったので、 [Quora](http://www.quora.com/) で聞いてみた所、これを教えてもらったので使ってみました。

ドキュメントが充実しておらず、結構苦労したので、Getting Start 的なものを書いてみました。（Eclipse の場合です）

1. microlog4android のサイトから microlog4android-1.0.0.jar をダウンロードする。2011.01.19追記：ログに出力されるログを日本時間で出力するよう改造してみました。下の GitHub に置いてある完成品をどうぞ。
2. 自分の Android プロジェクトに、jar を取り込む。
3. AndroidManifest.xml に android.permission.WRITE_EXTERNAL_STORAGE を追加する。（ログを SD-card に書きだす為に）
4. メインの Activity に以下のように記述する。

Most simple example of microlog4android

```java
// singleton logger object
static public Logger logger = LoggerFactory.getLogger();

// Called when the activity is first created.
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);

    // initialize logger
    logger.setLevel(Level.INFO);
    // write to LogCat
    logger.addAppender(new LogCatAppender());
    // write to text file of SD-card.(need WRITE_EXTERNAL_STORAGE permission)
    FileAppender fileAppender = new FileAppender();
    fileAppender.setAppend(true);
    fileAppender.setFileName("microlog4android.log");
    logger.addAppender(fileAppender);

    // test logging
    logger.debug("debug");  // <- no logging by level
    logger.info("information");
    logger.warn("warning");
    logger.error("error");
    logger.fatal("fatal");
}
```

とりあえずこれで、アプリを実行すると LogCat と SD-card（ルートの microlog4android.log）にログが出力されます。

完成品をとりあえず [GitHub](https://github.com/amay077/microlog4androidSample) に置いておきますので、ご参考に。

レベルを INFO にしているので、logger.debug はログに出力されません。

ハマりがちなのは、setFileName でディレクトリを指定した場合、そのディレクトリが存在してないと IOException が出る事です。(むーなんか使いにくいな)

### 2010.12.27 追記
setFileName のハマりポイントもう一つ。 設定は、SD-card のルートからの相対パスで行います。フルパスだと IOException が出ます。 たとえば Xperia で SD-card の sample ディレクトリ内にログファイルを作りたい場合、 /sdcard/sample/log.txt は NG で、 /sample/log.txt もしくは sample/log.txt が OK です。

他にも Log4j みたく config ファイルから設定を読み込んだり、Formatter を定義してログに出力する項目をカスタマイズできるのですが、それはまたいつの日か。

### 2010.12.26 追記
書いた！ → microlog4android で PatternFormatter を使う