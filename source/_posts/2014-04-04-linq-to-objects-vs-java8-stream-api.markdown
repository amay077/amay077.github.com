---
layout: post
title: "LINQ to Objects と Java8-Stream API の対応表"
date: 2014-03-20 15:15
comments: true
categories: [C#, Java, LINQ]
---
Java8 で ``filter`` や ``map`` が使えるようになったー！
というわけで .NET の LINQ to Objects との対応表を作ってみました。
<!--more-->
* LINQ - [Enumerable クラス (System.Linq)](http://msdn.microsoft.com/ja-jp/library/system.linq.enumerable(v=vs.110).aspx)
* Java8 - [Stream (Java Platform SE 8 )](http://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)

の比較です。

Java の方は

* [Collectors (Java Platform SE 8 )](http://download.java.net/jdk8/docs/api/java/util/stream/Collectors.html)

も使います。

まだ試したものは少ないので間違ってるかもしれない ＆ カテゴライズが適当 なので、編集リクエストしてもらえるとありがたいです。

| 機能 | LINQ | Java8 |
|----------------------|-------------------|----|
| **【基本的なやつ】** ||
| 抽出 | Where | filter 
| 射影 | Select | map 
| 並べ替え | OrderBy / OrderByDescending | sorted 
| 後続を並べ替え | ThenBy / ThenByDescending | n/a 
| 平坦化して射影 | SelectMany | flatMap 
| **【抽出系】** |  |  
| ｎ件飛ばす | Skip | skip 
| 条件を満たすまで飛ばす | SkipWhile | n/a 
| ｎ件まで流す | Take | limit 
| 条件を満たすまで流す | TakeWhile | n/a 
| **【合成系】** |  |  
| 連結 | Concat | concat 
| 積集合 | Intersect | n/a 
| 和集合 | Union | n/a 
| 差集合 | Except | n/a 
| 内部結合 | Join | n/a 
| 外部結合| GroupJoin | n/a 
| 並びを逆にする | Reverse | n/a 
| 2つの値を揃えて流す | Zip | n/a 
| **【グループ化、集計系】** |  |  
| 重複を無くす | Distinct | distinct 
| 畳み込み | Aggregate | reduce 
| グループ化 | GroupBy | Collectors.groupingBy 
| 平均 | Average | IntStream.average /  Collectors.summarizingXXX 
| 件数 | Count / LongCount | count 
| 最大 | Max | max 
| 最小 | Min | min 
| 合計 | Sum | IntStream.sum / Collectors.summarizingXXX
| 先頭 | First / FirstOrDefault | findFirst 
| 終端 | Last / LastOrDefault | n/a 
| とりあえず値を得る | | findAny 
| 集計用の汎用関数？ | | collect 
| 1件の値を得る | Single / SingleOrDefault | 
| 空なら既定値を返す | DefaultIfEmpty | 
| 全データが条件にマッチするか？ | All | allMatch 
| いずれかのデータが条件にマッチするか？ | Any | anyMatch 
| いずれかのデータも条件にマッチしないか？ | | noneMatch 
| **【生成系】** |  |  
| 空っぽ | Empty | empty 
| 範囲を生成 | Range | n/a 
| 繰り返す | Repeat | n/a 
| 無限リスト生成 | | generate / iterate 
| **【その他】** |  |  
| | SequenceEqual | 
| 列挙 | ToList().ForEach | forEach 
| なんか Action を挟む(デバッグ用？) | | peek 

ううむ、合成系の機能はほとんどないようです…ので自力でやるしか。
以下、サンプル。


## サンプル

LINQ の方は Mac+Mono(Xamarin) で試しています（ぼそり

### 抽出(Where)、並べ替え(OrderBy)、射影(Select)

0〜9 を、偶数値だけ抽出して、降順にソートして、値を10倍して、出力。

```csharp C#
Enumerable.Range(0, 10)
  .Where(x => x % 2 == 0)
  .OrderByDescending(x => x)
  .Select(x => x * 10)
  .ToList().ForEach(Console.WriteLine);
```

```java Java
Arrays.asList(0,1,2,3,4,5,6,7,8,9).stream()
  .filter(x -> x % 2 == 0)
  .sorted((x, y) -> y - x)
  .map(x -> x * 10)
  .forEach(System.out::println);
```

```
80 60 40 20 0
```

### 平坦化して射影(SelectMany)

1〜5のリストから、「n×10から始まるn件」のリストを生成。(結果見たほうが分かりやすいな（^_^;)

```csharp C#
Enumerable.Range(1, 5)
  .SelectMany(x => Enumerable.Range(10 * x, x))
  .ToList().ForEach(Console.WriteLine);
```

```java Java
Arrays.asList(1,2,3,4,5).stream()
  .flatMap(x -> IntStream.range(x * 10, x * 10 + x).boxed())
  .forEach(System.out::println);
```

```
10 
20 21 
30 31 32 
40 41 42 43 
50 51 52 53 54
```

### 抽出系(Take, Skip)

1〜10のリストの3件飛ばして、5件取得。

```csharp C#
Enumerable.Range(1, 10)
  .Skip(3)
  .Take(5)
  .ToList().ForEach(Console.WriteLine);
```

```java Java
// 無限リストでも limit あるから大丈夫
Stream.iterate(1, x-> x++)
  .skip(3)
  .limit(5)
  .forEach(System.out::println);
```

```
4 5 6 7 8
```

LINQ には件数でなく条件を指定できる ``TakeWhile`` ``SkipWhile`` がありますが、Java にはなさそうなので ``filter`` で代用しないといけなさそう。

```csharp C#
Enumerable.Range(1, 10)
  .SkipWhile(x => x < 4)
  .TakeWhile(x => x < 9)
  .ToList().ForEach(Console.WriteLine);
```

### 連結(Concat)

2つのリストをつなげる

```csharp C#
new int[] { 1, 2, 3 }.Concat(new int[]{ 30, 20, 10 })
.ToList().ForEach(Console.WriteLine);
```

```java Java
Stream.concat(
  Arrays.asList(1,2,3).stream(), 
  Arrays.asList(30,20,10).stream())
.forEach(System.out::println);
```

なんで static メソッドやねん…。

```
1 2 3 30 20 10
```

### 積集合(Intersect)、和集合(Union)、差集合(Except)

積集合：2つのリストから重複をなくす。
和集合：2つのリストをマージする。
差集合：リスト1を基準にリスト2との差分を得る。

```csharp C#
var list1 = new int[]{1,2,3,4,5,6};
var list2 = new int[]{8,7,6,5,4};

list1.Intersect(list2)
  .ToList().ForEach(Console.WriteLine);
            
list1.Union(list2)
  .ToList().ForEach(Console.WriteLine);
            
list1.Except(list2)
  .ToList().ForEach(Console.WriteLine);
```

```java Java
// 自力で実現かよｗ
list1.stream().filter(x -> list2.stream().anyMatch(y -> y == x))
  .forEach(System.out::println);

Stream.concat(list1.stream(), 
  list2.stream().filter(x -> list1.stream().noneMatch(y -> y == x)))
  .forEach(System.out::println);

list1.stream().filter(x -> list2.stream().noneMatch(y -> y == x))
  .forEach(System.out::println);
```

```
4 5 6 // 積
1 2 3 4 5 6 8 7 // 和
1 2 3 // 差
```

### 内部結合(Join)

商品マスタと売上テーブルを INNER JOIN する的な。

```csharp C#
var master = new [] {
    new { Id = 1, Name = "Apple" },
    new { Id = 2, Name = "Grape" }
}; 

var sales = new [] { 
    new { Id = 1, Sales = 100 },
    new { Id = 2, Sales = 200 },
    new { Id = 2, Sales = 300 },
    new { Id = 3, Sales = 400 },
};
            
master.Join(sales, 
  outer=>outer.Id, 
  inner=>inner.Id, 
  (o, i) => new { o.Name, i.Sales })
.ToList().ForEach(Console.WriteLine);
```

```java Java
// 自力
List<Pair<Integer, String>> master = Arrays.asList(
  new Pair<>(1, "Apple"),
  new Pair<>(2, "Grape")
);

List<Pair<Integer, Integer>> sales = Arrays.asList(
  new Pair<>(1, 100),
  new Pair<>(2, 200),
  new Pair<>(2, 300),
  new Pair<>(3, 400)
);

master.stream()
  .flatMap(outer -> sales.stream()
    .filter(inner -> outer.getKey() == inner.getKey())
    .map(z-> new Pair<String, Integer>(outer.getValue(), z.getValue())))
  .forEach(System.out::println);
```

```
{ Name = Apple, Sales = 100 }
{ Name = Grape, Sales = 200 }
{ Name = Grape, Sales = 300 }
```

### 外部結合(GroupJoin)

商品マスタと売上テーブルを OUTER JOIN する的な。結合先のテーブルに行が見つからなかったものは null になる。

```csharp C#
var master = new [] {
    new { Id = 1, Name = "Apple" },
    new { Id = 2, Name = "Grape" },
    new { Id = 5, Name = "Orange" },
}; 

var sales = new [] {  // Orange は無い
    new { Id = 1, Sales = 100},
    new { Id = 2, Sales = 200},
    new { Id = 3, Sales = 400},
};
            
master.GroupJoin(sales, 
  outer=>outer.Id, 
  inner=>inner.Id, 
  (o, i) => new { o.Name, FirstOfSales = i.Select(
    x=>(int?)x.Sales).FirstOrDefault() }) // 無かったら null にしたいので null許容型にしてから FirstOrDefault
.ToList().ForEach(Console.WriteLine);
```

たぶん普通は First じゃなくて Sum とか使うんだろう。

```java Java
// これも自力
List<Pair<Integer, String>> master = Arrays.asList(
  new Pair<>(1, "Apple"),
  new Pair<>(2, "Grape"),
  new Pair<>(5, "Orange")
);

List<Pair<Integer, Integer>> sales = Arrays.asList(
  new Pair<>(1, 100),
  new Pair<>(2, 200),
  new Pair<>(2, 300),
  new Pair<>(3, 400)
);

master.stream().map(outer->new Pair<String, Optional<Integer>>(outer.getValue(), 
  sales.stream()
    .filter(inner->inner.getKey() == outer.getKey()) // Id でフィルタ
      .map(x->x.getValue()) // Sales だけに射影
      .findFirst())) // 同一Id中の先頭
  .forEach(System.out::println);
```

```
[.NET]
{ Name = Apple, FirstOfSales = 100 }
{ Name = Grape, FirstOfSales = 200 }
{ Name = Orange, FirstOfSales = } // 相手が居ないやつは null になる

[Java]
Apple=Optional[100]
Grape=Optional[200]
Orange=Optional.empty // Option だから empty になるのは良い
```

### 2つの値を揃えて流す(Zip)

２つのリストの値をひとつずつセットにして流す。

```csharp C#
var arr1 = new int[] { 1, 2, 3, 4, 5 };
var arr2 = new string[] { "hoge", "fuga", "piyo" };

arr1.Zip(arr2, (x, y) =>  new {x, y})
    .ToList()
    .ForEach(Console.WriteLine);
```

```java Java
// FIXME どうやるの？ Streams.zip はどこいった？
```

```
{ x = 1, y = hoge }
{ x = 2, y = fuga }
{ x = 3, y = piyo }
```

### 重複を無くす(Distinct)

重複する数値リストから重複をなくす。

```csharp C#
new int[]{1,3,4,3,2,4}
  .Distinct()
  .ToList().ForEach(Console.WriteLine);
```

```java Java
Arrays.asList(1,3,4,3,2,4).stream()
  .distinct()
  .forEach(System.out::println);
```

```
1 3 4 2
```

### 畳み込み

いろいろな集計の素、畳み込み。言語により fold とか reduce とか aggregate とか、いろいろな呼び名がありますね。
よい例が浮かなかったので Max を実装してみました。

```csharp C#
var max = new int[]{1,5,3,7,2,4}
    .Aggregate(Int32.MinValue, (x, y) => Math.Max(x, y));
Console.WriteLine(max);
```

```java Java
int max = Arrays.asList(1,5,3,7,2,4).stream()
  .reduce(Integer.MIN_VALUE, (x, y) -> Math.max(x, y));
System.out.println(max);
```

```
7
```

### グループ化

リストの要素をキーにしてグループ化する。Salesは合計を計算する。

```csharp C#
var sales = new [] { 
    new { Id = 1, Sales = 100 },
    new { Id = 2, Sales = 200 },
    new { Id = 2, Sales = 300 },
    new { Id = 3, Sales = 400 },
};
            
sales.GroupBy(x=>x.Id, (Id, groupedSales) => new {Id, 
    SumOfSales = groupedSales.Sum( element => element.Sales) // Sales は合計する
  }) 
  .ToList().ForEach(Console.WriteLine);
```

（LINQ ではありませんが、 ``List.LookUp`` を使って実現することもできるようです → [コメント:2014/03/22 00:29](http://qiita.com/amay077/items/9d2941283c4a5f61f302#comment-82388821b902ad7999b0)）

```java Java
// javafx に Pair があったので Tuple 代わりに使っちゃった
List<Pair<Integer, Integer>> list1 = Arrays.asList(
  new Pair<>(1, 100),
  new Pair<>(2, 200),
  new Pair<>(2, 300),
  new Pair<>(3, 400)
);

list1.stream().collect(Collectors.groupingBy(x -> x.getKey()))
  .entrySet().stream() // group化の結果が Map なので、エントリを Stream 化
  .map(x -> new Pair<Integer, Integer>(
    x.getKey(), // Key が Id に相当
    x.getValue().stream().collect(Collectors.summingInt(y->y.getValue())))) // Value が List なのでまた Stream 化して合計を得る
  .forEach(System.out::println);

// Collectors.groupingBy 使わずに Map.merge を使ったほうが分かりやすい気も。。。
list1.stream().collect(
  () -> new HashMap<Integer, Integer>(),
  (map, item) -> map.merge(item.getKey(), item.getValue(), (x, y) -> x + y), // 同じキーの値を加算してく
  (left, right) -> left.putAll(right))
  .forEach((k, v) -> System.out.println(k + ":" + v));
```

Java の方、カオスすぎる…。.NET の ``IGrouping`` を Map でやってるからだな。

```
[.NET]
{ Id = 1, SumOfSales = 100 }
{ Id = 2, SumOfSales = 500 } // ID=2 の Sales が合計されている
{ Id = 3, SumOfSales = 400 }

[Java]
1=100
2=500
3=400
```

### 合計(Sum)、最大(Max)、最小(Min)、平均(Average)、件数(Count)、先頭(First)、終端(Last)

集計いろいろ。

```csharp C#
var list1 = Enumerable.Range(0, 10);
Console.WriteLine("Sum={0}", list1.Sum());
Console.WriteLine("Max={0}", list1.Max());
Console.WriteLine("Min={0}", list1.Min());
Console.WriteLine("Count={0}", list1.Count());
Console.WriteLine("First={0}", list1.First());
Console.WriteLine("Last={0}", list1.Last());
Console.WriteLine("Average={0}", list1.Average());
```

```java Java
List<Integer> list1 = Arrays.asList(0,1,2,3,4,5,6,7,8,9);
IntSummaryStatistics stats = list1.stream().collect(Collectors.summarizingInt(x -> x)); // Max,Min,Count,Average が取得できる
System.out.println("Sum=" + stats.getSum());
System.out.println("Max=" + stats.getMax());
System.out.println("Min=" + stats.getMin());
System.out.println("Count=" + stats.getCount());
System.out.println("First=" + list1.stream().findFirst().orElse(-1)); // summarizing では取れない
System.out.println("Last=" + list1.stream().sorted((x,y) -> y-x).findFirst().orElse(-1)); // 微妙
System.out.println("Average=" + stats.getAverage());

System.out.println("Average=" + IntStream.range(0, 10).average()); // 型指定 Stream なら average, sum がある（結果は Option に包まれる）
```

```
Sum=45
Max=9
Min=0
Count=10
First=0
Last=9
Average=4.5
```


…疲れた。。。