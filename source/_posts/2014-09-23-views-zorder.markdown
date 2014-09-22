---
layout: post
title: "ボタンを重ねた時の ZOrder に関する Tips"
date: 2014-09-23 00:20:38 +0900
comments: true
categories: [Android]
---
Android 開発でボタンを意図的に重ねたい時ってあんまりないんですけど、業務アプリなんか作ってますとたまにありまして。
<!--more-->
具体的には、

* [デザインの勉強にもなる、CSSで実装するパンくずのまとめ | コリス](http://coliss.com/articles/build-websites/operation/css/10-css-breadcrumbs.html)

みたいなパンくずリストを作りたい時。変な形のボタンを作るのは骨が折れるので、ボタンを重ねて、左側の方が手前に表示されるようにしたいわけです。(なぜそうしたいかはたぶん伝わらないので割愛)

つまりはボタンを重ねた時の ZOrder（Z-index）を制御したいわけです。

でいろいろトライ。

## LinearLayout の場合

LinearLayout で横並びにする場合。

```xml main.xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="30dp"
    android:orientation="horizontal">

    <Button
        android:background="#FF0000"
        android:text="AAA"
        android:layout_width="50dp"
        android:layout_height="match_parent" />
    <Button
        android:background="#00FF00"
        android:text="BBB"
        android:layout_marginLeft="-10dp"
        android:layout_width="50dp"
        android:layout_height="match_parent" />
    <Button
        android:background="#0000FF"
        android:text="CCC"
        android:layout_marginLeft="-10dp"
        android:layout_width="50dp"
        android:layout_height="match_parent" />
</LinearLayout>
```

### 結果

![](https://dl.dropboxusercontent.com/u/264530/qiita/zorder_test_01.png)


だめだー。
LinearLayout の Zorder は、並び順と連動してしまうので、右（若しくは下）ほど手前になってしまいます。

ちなみに、「``view.bringToFront()`` を叩けばいいんじゃね？」と思い、``buttonA.bringToFront()`` を実行すると、

じゃん↓

![](https://dl.dropboxusercontent.com/u/264530/qiita/zorder_test_02.png)

見事に AAA が右端にいったｗ

## RelativeLayout の場合（その１）

普通に RelativeLayout で、「BはAの右、CはBの右」と制約をつけてみます。

```xml main.xml
<RelativeLayout
    android:layout_width="match_parent"
    android:layout_height="30dp">

    <Button
        android:id="@+id/buttonA"
        android:background="#FF0000"
        android:text="AAA"
        android:layout_width="50dp"
        android:layout_height="match_parent" />
    <Button
        android:id="@+id/buttonB"
        android:layout_toRightOf="@+id/buttonA"
        android:background="#00FF00"
        android:text="BBB"
        android:layout_marginLeft="-10dp"
        android:layout_width="50dp"
        android:layout_height="match_parent" />
    <Button
        android:id="@+id/buttonC"
        android:layout_toRightOf="@+id/buttonB"
        android:background="#0000FF"
        android:text="CCC"
        android:layout_marginLeft="-10dp"
        android:layout_width="50dp"
        android:layout_height="match_parent" />
</RelativeLayout>
```

### 結果

![](https://dl.dropboxusercontent.com/u/264530/qiita/zorder_test_03.png)

んんー、まだダメかー。

## RelativeLayout の場合（その２）

その１の制約はそのままに、XML上での並び順を C、B、A に変えてみましょう。

```xml main.xml
<RelativeLayout
    android:layout_width="match_parent"
    android:layout_height="30dp">

    <Button
        android:id="@+id/buttonC"
        android:layout_toRightOf="@+id/buttonB"
        android:background="#0000FF"
        android:text="CCC"
        android:layout_marginLeft="-10dp"
        android:layout_width="50dp"
        android:layout_height="match_parent" />
    <Button
        android:id="@+id/buttonB"
        android:layout_toRightOf="@+id/buttonA"
        android:background="#00FF00"
        android:text="BBB"
        android:layout_marginLeft="-10dp"
        android:layout_width="50dp"
        android:layout_height="match_parent" />
    <Button
        android:id="@+id/buttonA"
        android:background="#FF0000"
        android:text="AAA"
        android:layout_width="50dp"
        android:layout_height="match_parent" />
</RelativeLayout>
```

### 結果

![](https://dl.dropboxusercontent.com/u/264530/qiita/zorder_test_04.png)

よしっ！期待した表示になりました。

## まとめ

　総合しますと、ZOrder は、LinearLayout でも RelativeLayout でも、XMLでは後で記述したものが手前になります。

　LinearLayout は、上から下、または左から右に並べるしかできないので、それに逆らうような ZOrder は付けられません。

 RelativeLayout は、制約に基づき描画されるので、XMLの記述順を工夫することで ZOrder をある程度コントロールできます。

以上、誰得Tips でした。

冒頭のようなパンくずリストをAndroidで作る方法教えてください。。。(FragmentBreadCrumbs もパンくずっぽくないじゃないですかぁ)
