---
layout: post
title: "ARC のメモリ解放タイミングを調べた"
date: 2013-09-02 21:55
comments: true
categories: [Objective-C,iOS]
---
一つの関数内で容量の大きなファイルを読み込み加工する処理を連続して行っていたらメモリが足りなくなった。
<!--more-->
ARC ではスコープを外れ(て参照カウンタがゼロになっ)たオブジェクトは、すぐに破棄されると思っていたのでしばらくハマった。

## 問題のソース(ARC使用)

ローカルでもWebでも何でもいいけど、ファイルから無視できない程度の容量のデータの読み込みを繰り返す処理。

```obj-c
- (IBAction)buttonDownWithArc:(id)sender {
    NSString* path = @".../bigdata.img";

    for (int i = 0; i < 10000; i++) {
        NSData* data = [NSData dataWithContentsOfFile:path];
        [NSThread sleepForTimeInterval:0.5];
        data = nil;
    }
}
```

これを Instruments でプロファイルするとこうなる。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/arc_memory_release_timing_01.png)

じゃんじゃんメモリ確保してしまう（汗
ARC で ``data`` は ``nil`` にしてるからスコープ外れた時にメモリ解放されると思っていたのだが。

ちなみにこの関数の処理が終了すると、メモリが解放される。

## 非ARC でやってみた

メモリ管理をマニュアルでやったらどうなるかを確認した。

```obj-c
- (IBAction)buttonDownNoArc:(id)sender {
    NSString* path = @".../bigdata.img";

    for (int i = 0; i < 10000; i++) {
        NSData* data = [NSData dataWithContentsOfFile:path];
        [NSThread sleepForTimeInterval:0.5];
        [data dealloc];
        data = nil;
    }
}
```

この時のメモリ確保状況は、期待した通りになった。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/arc_memory_release_timing_02.png)

メモリ使用量が線形に**増えない**ことが分かる。ARC 利用時にもこうなるようにしたい。

状況は、スコープ内変数の破棄が、関数を抜ける時に遅延されている。
ARC 周りの情報をいろいろ漁っていて、AutoReleasePool との関わりが怪しいと予想した。

* [[iOS5] ARC (Automatic Reference Counting) : Overview - iOS 開発ブログ Natsu's note ](http://blog.natsuapps.com/2011/11/ios5-arc-overview.html)

より引用：

> ###retain, release, autorelease, deallocはコンパイラのお仕事
>
>ARCを利用する場合、コンパイラが
>
> * retain, release, autoreleaseを挿入してくれる（自分で呼んではいけない。コンパイラエラーになる）。
> * deallocを適切な位置に挿入してくれる（deallocのオーバーライドは可能。ただし[super dealloc]は不可能）。
>
>ことになります。

コンパイラにより関数単位で ``@autoreleasepool {  }`` が挿入されているとしたら、最初の図のような動きになるはず。ということは、for ループの中に @autorelease を持ってったらどうか？

## ARC + @autoreleasepool 版

for の中の処理を ``@autoreleasepool { }`` で括ってみた。

```obj-c
- (IBAction)buttonDownWithArcAndAutoRelease:(id)sender {
    for (int i = 0; i < 100; i++) {
        @autoreleasepool {
            NSData* data = [NSData dataWithContentsOfFile:_path];
            [NSThread sleepForTimeInterval:0.5];
        }
    }
}
```

すると、

![img](https://dl.dropboxusercontent.com/u/264530/qiita/arc_memory_release_timing_03.png)

やたー、期待する動きになったぞ。

## まとめ

とここまで調べて、しばらく Obj-C さわってなかったので埃をかぶっていた

* [エキスパートObjective-Cプログラミング -iOS/OS Xのメモリ管理とマルチスレッド-](http://www.amazon.co.jp/gp/product/4844331094?ie=UTF8&camp=1207&creative=8411&creativeASIN=4844331094&linkCode=shr&tag=oku2008-22)

を引っ張り出してきて読んだら、P.25 にまさにその事が書かれていて泣いた。

>とはいえ、autorelease されたオブジェクトが大量に発生した場合、NSAutoReleasePool のオブジェクトが破棄されない限り、それらのオブジェクトは release されないので、メモリ不足に陥る場合があります。典型的な例は、大量の画像をリサイズしながら読み込む場合でしょう。…
>
>
>        for (int i = 0; i < 画像数; i++) {
>            /*
>             * 画像読み込み処理
>             * autoreleaseされたオブジェクトが大量発生。
>             * NSAutoReleasePoolのオブジェクトが破棄されないため
>             * いずれメモリ不足発生！
>             */	       
>        }
>

勉強しなおします。。。