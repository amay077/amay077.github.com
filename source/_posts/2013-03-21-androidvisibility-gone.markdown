---
layout: post
title: "android:visibility の gone について調べてみた"
date: 2010-12-20 10:04
comments: true
categories: [Android, Java]
---
Android の UI レイアウト作成で、コンポーネントの表示・非表示についての visibility プロパティについて調べてみました。
<!--more-->

visibility は[リファレンス](http://developer.android.com/reference/android/view/View.html#attr_android:visibility)によると、visible / invisible / gone の３種類があるとのこと。

自分は VB6 とか C# 出の人間なので、「なんで２種類(Boolean)じゃないんだろう」と思っていましたので、ためしにやってみました。

レイアウトにボタン（じゃなくても良いけど）を２つ貼り、一つ（A とします）を可変幅に、もう一つ（B とします）を固定幅にしました。
ボタンBは常に固定幅で画面右にあって、ボタンAは残った領域に fill するイメージです。
（※ボタンA の layout_weight を B より大きな値にセットするのがミソです） 

この状態で、ボタン B の visibility を変えてみて、どんな変化が起こるか確認してみました。

それが↓これ。

!["1"](https://dl.dropbox.com/u/264530/qiita/androidvisibility_1.png)

なるほど。
invisible は、そこに存在するけど見えていない。ので空白の領域ができる。
gone は、存在自体が無くなる。ので ボタンA がいっぱいまで表示される。
VB6 や C# に例えていうなら、Visible の true が "visible" で、false は "gone" なんですね、納得。 

 

ちなみに "gone" を[英和辞典で調べる](http://eow.alc.co.jp/gone/)と、「消失した、いなくなった」 という意味もあるそうで、なるほど。