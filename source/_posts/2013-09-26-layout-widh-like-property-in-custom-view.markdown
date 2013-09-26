---
layout: post
title: "自作ビューで layout_width のマネをしたい時"
date: 2013-09-26 23:55
comments: true
categories: [Android]
---
``android:layout_width`` って、型は ``dimension`` であるのに、Layout XML には、 ``"match_parent"`` とか ``"wrap_content"`` とか指定できる。これと同じことを自作ビューの自作プロパティでやりたい。
<!--more-->
自作ビュー(カスタムビュー)の作りかたは下などを参照。

* [Androidでxmlファイルを用いてカスタムViewを作る方法 | Tech Booster](
http://techbooster.org/android/application/7361/)

で、``android:layout_width`` のマネをするには、 attrs.xml に以下のように書く。

```xml attrs.xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
  <declare-styleable name="MyCustomView">
    <attr name="my_width" format="dimension">
      <enum name="fill_parent" value="-1" />
      <enum name="match_parent" value="-1" />
      <enum name="wrap_content" value="-2" />
    </attr>
  </declare-styleable>
</resources>
```

Android Framework のソースコードにこう書いてあったので、真似しただけ。

* [GrepCode: frameworks / base / core / res / res / values / attrs.xml - Source Code View](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/4.3_r2.1/frameworks/base/core/res/res/values/attrs.xml)

自作ビューの実装クラスで、この値を読み込む時は、``TypedArray.getLayoutDimension`` を使う。

```java MyCustomView.java
public MyCustomView(Context context, AttributeSet attrs) {
    super(context, attrs);
    
    TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.MyCustomView, 0, 0);

    _myWidth = a.getLayoutDimension(R.styleable.MyCustomView_my_width, 
            LayoutParams.WRAP_CONTENT);
    
    a.recycle();
}
```

Android Framework 側で仕様が増えた時は、追従しないといかんのかー。