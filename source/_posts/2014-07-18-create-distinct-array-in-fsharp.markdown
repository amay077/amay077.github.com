---
layout: post
title: "ランダムで且つ重複しない数値リストを作る"
date: 2014-07-08 00:00
comments: true
categories: [F#, Xamarin]
---
F# 入門中です。
<!--more-->

[フラッシュ暗算](http://www.shuzan.jp/kentei/flash/)ぽいアプリを作っていて、次々と表示する数値は、

* ランダムで
* 一度使った数値は二度と使わなくて
* ０も使わない

というルールにしています（公式ルールは知らない）

``makeRandomList`` の引数 ``rand`` は ``System.Random`` のインスタンス、``count`` は生成するリストの要素数、``arr`` は生成した数値群(=再帰処理で値の既出判定に使う)としています。

``rand.Next(10)`` で得た値が、0 もしくは ``arr`` に存在する場合は、もう一度同じパラメータで再帰呼び出し、そうでない場合は ``count`` を減算しつつ、``arr`` に値を連結して再帰呼び出しします。``count`` が ``0`` になったら ``arr`` を返して終わります。

```fsharp
let rec makeRandomList = fun (rand:Random, count:int, arr:List<int>) ->
    match count with
    | 0 -> arr
    | _ -> 
        let n = rand.Next(10) // 1桁の数値
        if n = 0 || arr |> List.exists (fun x->x=n) then 
            makeRandomList (rand, count, arr) 
        else makeRandomList (rand, count-1, [n] @ arr)

let arr = makeRandomList (new Random(DateTime.Now.Millisecond), 3, [])
for x in arr do
    Console.WriteLine x

// 出力例
7
5
4
```

こんな感じで書けばいいのかなあ。
なんかところどころ手続き型の書き方になってる気がしますが、C# で ``while`` で処理するよりはスマートにできた気がします。
