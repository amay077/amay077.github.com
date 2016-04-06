---
layout: post
title: "RxJava でズンドコキヨシ(window or buffer 使用)"
date: 2016-03-13 23:59:59 +0900
comments: true
categories: [RxJava, Java, ReactiveX, ズンドコキヨシ]
---

調子に乗って RxJava でもやってみた。
<!--more-->

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">Javaの講義、試験が「自作関数を作り記述しなさい」って問題だったから<br>「ズン」「ドコ」のいずれかをランダムで出力し続けて「ズン」「ズン」「ズン」「ズン」「ドコ」の配列が出たら「キ・ヨ・シ！」って出力した後終了って関数作ったら満点で単位貰ってた</p>&mdash; てくも (@kumiromilk) <a href="https://twitter.com/kumiromilk/status/707437861881180160">2016年3月9日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

* [C# と Reactive Extensions でズンドコキヨシ](http://qiita.com/amay077/items/85dfc4bd194f57c52c57)


がんばって [``Observable.window``](http://reactivex.io/documentation/operators/window.html) を使ってみた。

```java zondoko.java

// なんちゃってラムダ使用。あと Android。
public void doZondoko() {
    final Random random = new Random();
    final List<String> PATTERN = Arrays.asList("ずん", "ずん", "ずん", "ずん", "どこ");
    final String K = "キ・ヨ・シ！";

    Observable.concat( // ※ の Observable<List<String>> を直列に連結
        Observable.interval(500, TimeUnit.MILLISECONDS)
            .map(_ -> random.nextInt(2) == 0 ? "ずん" : "どこ") // ランダムに ずん or どこ
            .window(PATTERN.size(), 1) // 要素数5のWindowを1ずつズラしてく
            .map(window -> window.toList())) // Observable<Observable<String>> を Observable<List<String>> に変換 ※
        .flatMap(window -> {
            if (sequenceEqual(window, PATTERN)) { // パターンと一致していたら…
                final List<String> says = new ArrayList<>();
                says.addAll(window);
                says.add(K);                      // キ・ヨ・シ！を追加
                return Observable.concat(
                        Observable.just(says),
                        Observable.just(Collections.<String>emptyList())); // 終了判定用の空リスト
            } else {
                return Observable.just(window);
            }
        })
        .takeWhile(says -> !says.isEmpty())  // 空リストになるまで繰り返す
        .subscribe(says -> Log.d(TAG, dump(says)));
}

/** リストとリストの要素一致 */
private boolean sequenceEqual(List<String> listA, List<String> listB) {
    Iterator<String> iterA = listA.iterator();
    Iterator<String> iterB = listB.iterator();

    while (iterA.hasNext() && iterB.hasNext()) {
        if (iterA.next() != iterB.next()) {
            return false;
        }
    }
    return (!iterA.hasNext() && !iterB.hasNext());
}

/** リスト内容をダンプ */
private String dump(List<String> list) {
    final StringBuilder b = new StringBuilder();
    for (String s : list) {
        if (!TextUtils.isEmpty(b.toString())) {
            b.append(", ");
        }
        b.append(s);
    }

    return b.toString();
}
```

> どこ, ずん, どこ, どこ, ずん<br/>
ずん, どこ, どこ, ずん, どこ<br/>
どこ, どこ, ずん, どこ, ずん<br/>
どこ, ずん, どこ, ずん, どこ<br/>
ずん, どこ, ずん, どこ, どこ<br/>
どこ, ずん, どこ, どこ, ずん<br/>
ずん, どこ, どこ, ずん, ずん<br/>
どこ, どこ, ずん, ずん, ずん<br/>
どこ, ずん, ずん, ずん, ずん<br/>
ずん, ずん, ずん, ずん, どこ, キ・ヨ・シ！

「window(5, 1) -> toList -> concat してるならそれは ``buffer(5, 1)`` やんけ」というのを [こちら](http://qiita.com/do6gop/items/c4941f6fb2bdc1c0c0f1) で知って、 ``buffer`` 版も書いてみた。

```java Zondoko_buffer.java
public void doZondoko() {
    final Random random = new Random();
    final List<String> PATTERN = Arrays.asList("ずん", "ずん", "ずん", "ずん", "どこ");
    final String K = "キ・ヨ・シ！";

    Observable.interval(500, TimeUnit.MILLISECONDS)
        .map(_ -> random.nextInt(2) == 0 ? "ずん" : "どこ") // ランダムに ずん or どこ
        .buffer(PATTERN.size(), 1) // 要素数5のBufferを1ずつズラしてく
        .flatMap(buf -> {
            if (sequenceEqual(buf, PATTERN)) { // パターンと一致していたら…
                final List<String> says = new ArrayList<>();
                says.addAll(buf);
                says.add(K);                      // キ・ヨ・シ！を追加
                return Observable.concat(
                        Observable.just(says),
                        Observable.just(Collections.<String>emptyList())); // 終了判定用の空リスト
            } else {
                return Observable.just(buf);
            }
        })
        .takeWhile(says -> !says.isEmpty())  // 空リストになるまで繰り返す
        .subscribe(says -> Log.d(TAG, dump(says)));
}
```


* [さまざまなズンドコキヨシ](http://qiita.com/B73W56H84/items/519e27a1aed5e6d5304f#%E3%81%82%E3%82%8F%E3%81%9B%E3%81%A6%E8%AA%AD%E3%81%BF%E3%81%9F%E3%81%84)
* [ズンドコキヨシまとめ](http://qiita.com/shunsugai@github/items/971a15461de29563bf90)
