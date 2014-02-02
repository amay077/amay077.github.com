---
layout: post
title: "Objective-C と Java と C# でクロージャ的な書き方の比較"
date: 2014-01-29 15:52
comments: true
categories: [Java, Xamarin, C#, Objective-C]
---

クロージャとかラムダとか匿名** とか名前はいろいろですけど、各言語の書き方と動き（特に変数の扱い）について比べてみました。

ついでに非同期処理の例にもなってしまいました。
<!--more-->
### Objective-C

Blocks を使います。

```objc Obj-C
int x = 1;
__block int y = 2;

dispatch_queue_t q_global = dispatch_get_global_queue(0, 0);
dispatch_async(q_global, ^{
    x = 10; // できない(コンパイルエラー
    y = 20; // できる
    
    int z = x + y;
    
    [self dispValue:z]; // self の参照カウンタが+1される
});
```

普通に宣言した変数を Block の中で使うと、自動的に「キャプチャ」され、変数の複製される。この変数には、 Block 内では代入できずコンパイルエラーとなる。
``__block`` を付けた変数は、Block 内外で同じ実体を参照でき、代入もできる。
``self`` やプロパティを Block 内で使用すると参照カウンタがインクリメントされ、明示的に release しないとリークする。
あるいは、Block 外で ``__weak`` を付けた変数に代入しておくと、これは参照カウンタがインクリメントされない。


### Java 6 (Android ベースなので…)

匿名クラスです。

```java Java
int x = 1;
final int y = 2;

ExecutorService executor = Executors.newSingleThreadExecutor();
executor.submit(new Runnable() {
	@Override
	public void run() {
		y = 20; // できない(コンパイルエラー
		int z = x + y; // できない(コンパイルエラー
		
		String typeName = this.getClass().getInterfaces()[0].getName(); // Runnable になる
	}
});
```

Java は匿名クラスの実装中に使える変数はかなり制限がある。
普通に宣言した変数は、匿名クラス内では使えない(コンパイルエラー)。
``final`` を付けて宣言した変数は、匿名クラス内では参照のみ可能。ちょうど Objective-C の通常変数を Block 内で使った時と同じ。
Obj-C の ``self`` にあたる ``this`` は匿名クラス内では、その匿名クラスを示す。

### C＃

ラムダ式です。

```csharp C#
int x = 1;
const int y = 2;
Task.Factory.StartNew(() => 
{
    x = 10; // OK
    y = 20; // これはダメ、const だから。
    var z = x + y;

    var typeName = this.GetType().Name;
});
```

C# はかなりゆる〜い印象。
普通に宣言した変数を、ラムダ式の中でも自由に read/write できてしまう。write できちゃうのはこわい。
``this`` は、ラムダ式の外側のクラスを示す。
　

## 所感

個人的には、Java のガチガチなのが好きかも。できる事が限定されているのでミスしにくい。
Objective-C は、ローカル変数は良いけど、self とか使っちゃうミス起こしそう。
C# は、普通に書き換えられて超不安、という感じ。なるべく const 使うようにしたい。
　
　
　

しかし記述量は C# が一番少ないですね、Xamarin いいよ Xamarin。。。