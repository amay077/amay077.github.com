---
layout: post
title: "Cold を Hot にできる。そう、Publish ならね。"
date: 2012-10-03 19:13
comments: true
categories: [Java, ReactiveExtensions, reactive4java, Android]
---
Rx いいよ Rx とか言っておきながら、いままで Cold と Hot の違いについて、ちゃんとわかってませんでしたスイマセン。

<!--more-->

そのため、

* [reactive4java で端末の方位を取得しつづける - Qiita](http://qiita.com/items/07762776102dbc84b1c7)
* [reactive4java で位置を取得し続ける - Qiita](http://qiita.com/items/e15ba88d51938531b1a3)

で作ったサンプルプログラム、盛大にバグってましたorz

## Cold な Observable と Hot な Observable
音楽プレーヤ iPod に例えると自分なりにしっくり来ました。
Observable が iPod で、Observer は聴く人。
なんとこの iPod はイヤホンジャックがたくさんあります。
そしてさらにこの iPod はイヤホンを接続するだけで再生が始まり、抜くと停止します。

Cold な iPod では、聴く人がイヤホンを接続すると、音楽が最初から再生されます。
次の人がイヤホンを接続すると、また音楽が最初から再生されます。(最初の人は、再生されなおすわけじゃないよ。)

Hot な iPod では、最初に聴く人がイヤホンを接続すると、音楽が再生され始めます。
次の人が、イヤホンを接続すると、その人は途中から聴くことになります。一つのストリーミング放送をみんなで聴くみたいな。
最後の人がイヤホンを外すと、再生が止まります(この辺はストリーミングと微妙に感覚が異なる、ストリーミングって、聴いてる人が居ようが居まいが流され続けるってイメージだから)。

## 何が問題か

以上を踏まえた上で、[reactive4java で位置を取得し続ける - Qiita](http://qiita.com/items/e15ba88d51938531b1a3) で作ったプログラムのどこが問題だったかというと。

位置を聞くために接続をすると、その都度、``locMan.requestLocationUpdates`` が呼ばれているという事です。上記の音楽プレーヤで例えると、``player.start()`` です。

位置情報の取得は、ホントに無限ストリーミングなので、一見問題無さげに見えますが、２つリスナを登録するのは頂けません。(中には複数のリスナを登録できない API もあるでしょう、ありました。それで気づいたんです。)

これは最初の register でのみ ``locMan.requestLocationUpdates`` が実行され、2番目以降の register では、observer.next だけが呼ばれるようにしないといけません。
そのためには、register した複数の observer を保持・管理する必要があります。うげー。

## そこで Publish ですよ。
.publish() を Observable のおしりにくっつけます。はい、これだけ。本当に。簡単すぎて「いいの？」って思っちゃうくらい。

``java getCurrectLocationAsHotObservable.java
/**
 * 位置を取得し続ける(Hot)
 */
public static ObservableBuilder<Location> getCurrentLocationAsHotObservable(          
	final Context context, final String provider) {
    return ObservableBuilder.from(
		getCurrentlocationAsObservable(context, provider)
		).publish(); // Cold → Hot へ変換！
}
```

これだけで、複数の人が位置を聴きに来ても、``locMan.requestLocationUpdates`` が呼ばれるのは１回だけである、Hot な Observable になります。Rx すげえよ Rx！

reactive4java のソースを読んだところ、前述の「複数の observer を保持・管理して、最初だけリスナ登録して、誰もいなくなったらリスナ解除する」みたいな面倒なことを publish の中(正確には observeOn)で行なってくれているようです。

ますます Reactive Extensions が好きになりましたよ。

## .NET の Reactive Extensions と reactive4java の違い
.NET の Reactive Extenstions では、Publish は、``IConnectableObservable<T>`` を返すそうです。そして ``Subscribe`` しただけでは再生は開始されず、``Connect`` した時に再生されるのだ、とも。

一方、[reactive4java](http://reactive4java.googlecode.com/svn/trunk/Reactive4Java/docs/javadoc/index.html) では、``publish`` は、普通の ``Observable<T>`` を返します。Observable には register(.NET の Subscribe に相当)しかないので、これを呼び出した時に再生が開始されます。

特に reactive4java の挙動で困っていませんが、今後のバージョンアップで .NET 側に合わせられるかも知れません。
ConnecableObservable という interface は既に用意されていますが、使われていないようです。

## 参考
* [Rx入門 (14) – Cold to Hot変換 : xin9le note](http://xin9le.net/archives/104)
* [Reactive Extensions再入門 その３６「ColdからHotへ！Publishメソッドと参照カウンタ？RefCountメソッド」 - かずきのBlog@Hatena](http://d.hatena.ne.jp/okazuki/20120212/1329059831)
* [neue cc - Reactive Extensions for .NET (Rx) メソッド探訪第7回:IEnumerable vs IObservable](http://neue.cc/2010/06/24_263.html)

Rx の神々のみなさんが懇切丁寧に解説してくださってるのに、失敗しないと気づかない自分のバカバカ！