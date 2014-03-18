---
layout: post
title: "ふたつの Iterator を LINQ で"
date: 2014-03-19 00:33
comments: true
categories: [C#, Java]
---
ふたつの Iterator を LINQ で

* [ふたつのIterator - プログラマーの脳みそ](http://d.hatena.ne.jp/Nagise/20140315/1394884271)

を拝見しました。
<!--more-->
コメントで「Zip じゃん」というのがあり、「確かに！」と思ったのでやってみました。

## C＃ の場合

```csharp
// using using System.Linq;

var arr1 = new int[] { 1,2,3,4,5 };
var arr2 = new string[] { "hoge", "fuga", "piyo" };

arr1.Zip(arr2, (x, y) =>  new {x, y})
    .ToList()
    .ForEach(Console.WriteLine);
```

#### 結果

```
{ x = 1, y = hoge }
{ x = 2, y = fuga }
{ x = 3, y = piyo }
```

うむ、シンプル。要素数が違ってても少ない方に合わせてくれます。

## Java の場合

[reactive4java](https://code.google.com/p/reactive4java/) というライブラリがありまして、これは Java で Reactive Extensions を実現するライブラリなのですが、LINQ的な機能が ``Interactive`` というクラスで提供されています。

これを使うと Zip ができます。

```java
//import hu.akarnokd.reactive4java.base.Func2;
//import hu.akarnokd.reactive4java.interactive.Interactive;

Iterable<Integer> array1 = Arrays.asList(1,2,3,4,5);
Iterable<String> array2 = Arrays.asList("hoge", "fuga", "piyo");

Iterator<Pair<Integer, String>> zippedIter = 
		Interactive.zip(array1, array2, 
				new Func2<Integer, String, Pair<Integer, String>>() {
	@Override
	public Pair<Integer, String> invoke(Integer x, String y) {
		return new Pair<Integer, String>(x, y);
	}
}).iterator();

while (zippedIter.hasNext()) {
	Pair<Integer, String> p = zippedIter.next();
	Log.d("StartupActivity", String.format("x=%d, y=%s", p.first, p.second));
}
```

ああ、Android で試したので ``Pair`` とか使ってしまった。
普通の Java の場合は自作の Tuple などに置き換えを。

#### 結果

```
x=1, y=hoge 
x=2, y=fuga 
x=3, y=piyo 
```

C# より冗長ですけど、いい感じで利用できるのではと思います。

reactive4java が Java8 のラムダ式に対応してくれると、上のコードはもっと簡潔に書けます。

[Reactive4Java8](https://code.google.com/p/reactive4java/wiki/Reactive4Java8) には、対応してる感じが見られますが、[Top ページ](https://code.google.com/p/reactive4java/) によると、どうやら、「reactive4java の開発は終了し、[RxJava](https://github.com/Netflix/RxJava) の開発に参加するつもりだ」みたいなことが書いてあります。

また、RxJava でなく reactive4java を使う利点として、``Interactive`` 機能群の存在を挙げています。
単純に LINQ 的な機能を Java で使いたいならば、 reactive4java はまだまだ役に立つと思います。

最後に、もしあなたが Android 開発をしていて、Java の冗長さに嫌気がさしているなら、[**Xamarin へどうぞ**](http://xamarin.com/csharp) 