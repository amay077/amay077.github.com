---
layout: post
title: "Swift で @autoreleasepool どうやるの？"
date: 2014-06-17 00:00
comments: true
categories: [Swift]
---
``autoreleasepool {  }`` だそうです。

<!--more-->

* [automatic ref counting - What is the equivalent of @autoreleasepool in Swift? - Stack Overflow](http://stackoverflow.com/questions/24152050/what-is-the-equivalent-of-autoreleasepool-in-swift)

こんな感じで使うようです。

```
for i in 1...10000 {
  autoreleasepool {
    // do heavy work
  }
}
```

[iOS - ARC のメモリ解放タイミングを調べた](http://qiita.com/amay077/items/95a4139e6f553d8a56a1) は Swift でも有効なようで。

~~http://swift-lang.org/tryswift/ で試してみたら "unexpected token: autoreleasepool" で怒られた。。。Xcode6 を入れないと動かせないんですかね。~~

