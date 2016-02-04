---
layout: post
title: "Android-Java でリスト操作するなら IxJava が決定版だと思う"
date: 2016-01-21 23:59:59 +0900
comments: true
categories: [Android, Java]
---
Android では Java8 が使えないため、 'Yet another Stream API' なライブラリがいくつかあります。
<!--more-->

* [Androidでリスト操作するなら、Lightweight-Stream-APIが便利 - Qiita](http://qiita.com/tsumuchan/items/8e438a2ea653fa786c23)
* [JavaにC#のLINQを移植してみた - jLinqer - Qiita](http://qiita.com/k--kato/items/ec7ab8b392fa8bb0a732)
* [What is the Java equivalent for LINQ? - Stack Overflow](http://stackoverflow.com/questions/1217228/what-is-the-java-equivalent-for-linq)

普段 C# を使っているため LINQ to Objects の便利さをなんとか Androidアプリ開発でも享受したい。
そこで個人的に「これだ！」と思っているライブラリが IxJava です。

* [akarnokd/ixjava: Interactive Extensions for Java](https://github.com/akarnokd/ixjava)

## IxJava とは？

一言でいえば *「LINQ to Objects の Java版」* です。

README より、

> Interactive Extensions for Java, the dual of RxJava. Originally implemented in the Reactive4Java framework, now converted to work with RxJava.
> 
> The aim is to provide pull-based datastream support with the same naming as in RxJava mainly for the pre-Java-8 world. 

開発者の akarnokd 氏は、 RxJava の登場以前から [Reactive4Java](https://code.google.com/p/reactive4java/) という「Java版Rx」を開発しており、これには大きく２つの機能が含まれていました。

* ``Reactive<T>`` : Reactive Extension の Java実装
* ``Interactive<T>`` : LINQ to Objects の Java実装

そう、 akarnokd 氏は、Rx と共に LINQ も Java に移植していたのです。
その後、彼は RxJava への参加を表明し、 reactive4java は開発終了となりましたが、RxJava には LINQ 相当の機能は含まれません。
そこで彼は、 ``Interactive<T>`` だけを *IxJava* として切り離し、純粋な *「LINQ to Object for Java」* として開発続行したのです。

akarnokd 氏は RxJava の [Contributors](https://github.com/ReactiveX/RxJava/graphs/contributors) を見ると中心的な開発者であると思われます。そんな彼が開発した ixjava も安心できる品質ではないかと思います。（ちょっと ixjava の知名度が低いのが残念ですが。ただ reactive4java の ``Interactive<T>`` を使ってきましたが問題はありません。）

## 使い方（Android の場合）

### 導入方法

Module の ``build.gradle`` に以下を追加するだけです。

```java
dependencies {
    compile "com.github.akarnokd:ixjava:0.90.0"
}
```

### 使用例

* [LINQ to Objects と Java8-Stream API の対応表 - Qiita](http://qiita.com/amay077/items/9d2941283c4a5f61f302)

のサンプルコードの一部を IxJava で書いてみました。

#### 抽出(filter)、並べ替え(orderBy)、射影(map)

```java ixjava
Ix.from(Arrays.asList(0, 1, 2, 3, 4, 5, 6, 7, 8, 9))
  .filter(x -> x % 2 == 0) // 可視性向上の為のなんちゃってラムダ
  .orderBy(x-> -x)  // OrderByDescending がないので
  .map(x -> x * 10)
  .toList();
```

```
//出力
80 60 40 20 0
```

#### 平坦化して射影(flatMap)

```java ixjava
Ix.from(Arrays.asList(1, 2, 3, 4, 5))
  .flatMap(x −> Ix.range(x * 10, x))
  .toList();
```

```
//出力
10 
20 21 
30 31 32 
40 41 42 43 
50 51 52 53 54
```

#### 2つの値を揃えて流す(zip)

Stream API には無いが IxJava にはあるのだよ。

```java ixjava
Ix.from(Arrays.asList(1, 2, 3, 4, 5))
  .zip(Ix.from(Arrays.asList("hoge", "fuga", "piyo")), 
    (x, y) -> new Pair<Integer, String>(x, y))
  .toList();
```

```
//出力
{ first = 1, second = hoge }
{ first = 2, second = fuga }
{ first = 3, second = piyo }
```

#### Ix<T> のメソッド一覧

あとはテキトーに抜き出したメソッド一覧を置いておきますね。
RxJava や LINQ とほとんど同じなのでだいたい想像付くと思います。
（何気に ``toObservable`` で ``Observable<T>`` にも変換できますね。）
あ、あとタイトルには Androidの〜 と書きましたが、普通の Java でもフツーに使えますので。

* aggregate
* all
* any
* argAndMax
* argAndMin
* averageBigDecimal
* averageBigInteger
* averageDouble
* averageFloat
* averageInt
* averageLong
* buffer
* call
* concat
* concatWith
* concatWithAll
* contains
* count
* countLong
* defer
* dematerialize
* distinct
* distinctNext
* doOnCompleted
* doOnNext
* doWhile
* empty
* endWith
* error
* filter
* filterIndexed
* first
* flatMap
* forEach
* from
* fromPart
* generate
* groupBy
* into
* isEmpty
* iterator
* join
* just
* last
* map
* mapIndexed
* materialize
* max
* maxBy
* mayBy
* memoize
* memoizeAll
* min
* minBy
* minxBy
* newBuilder
* ofType
* orderBy
* print
* println
* prune
* publish
* range
* removeAll
* repeat
* replay
* run
* scan
* share
* skipLast
* startWith
* subsequent
* sumBigDecimal
* sumBigInteger
* sumDouble
* sumFloat
* sumInt
* sumIntAsDouble
* sumLong
* sumLongAsDouble
* take
* takeLast
* toArray
* toBuilder
* toHashMap
* toHashMultimap
* toList
* toMap
* toMultimap
* toObservable
* zip

