---
layout: post
title: "hxLINQ というのがあったので使ってみた。"
date: 2012-10-03 17:17
comments: true
categories: [haxe, Linq]
---
hxLINQ というのがあったので使ってみた。
JSLinq をベースにして作られたものらしい。
Flash/JS/PHP/Neko/C++ でテスト済だそうです。

<!-- more -->

##導入
まず、 hxLINQ をインストールします。

	haxelib install hxLINQ

Sublime Text2 と [haxe-sublime2-bundle](https://github.com/clemos/haxe-sublime2-bundle) を使っている場合は、``shift+ctrl+L`` で ライブラリの一覧が出るのでそこから選択するだけです。ﾁｮｰｲｰﾈ!

##使い方

こんな感じで。

```as hxLINQExample.hx
var it = new hxLINQ.LINQ([1,2,3,4,5,6,7,8,9])
.where(function(v:Int, i:Int):Bool {
	return v % 2 == 0; // 抽出(偶数のみ)
})
.select(function(v:Int):Float {
	return v / 10.0; // 加工(10で除算)
})
.iterator();

while(it.hasNext()) {
	trace(it.next()); // out:0.2, 0.4, 0.6, 0.8
}
```

ひと通りの機能は揃ってるので使えそうな感じだけど、Rx の機能まで求めてしまうのは贅沢でしょうか。

##参考
* [andyli / hxLINQ](https://github.com/andyli/hxLINQ)
* [hxLINQ - lib.haxe.org](http://lib.haxe.org/p/hxLINQ)
