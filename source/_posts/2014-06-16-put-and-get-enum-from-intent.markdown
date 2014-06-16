---
layout: post
title: "Intent に Enum を詰めて、取り出す"
date: 2014-04-23 15:22
comments: true
categories: [Android]
---
タイトルの通りです。
<!--more-->

```java enum
public enum MyTypes {
	One,
	Two,
	Three
}
```


```java put
intent.putExtra("hoge",MyType.Two);
```

```java get
MyTypes t = (MyTypes)intent.getSerializableExtra("hoge");
```

取り出すのは ``getSerializableExtra`` であることに注意。

なんか int に変換されそうじゃん、と思って ``getIntExtra`` を使うと取り出せない。
