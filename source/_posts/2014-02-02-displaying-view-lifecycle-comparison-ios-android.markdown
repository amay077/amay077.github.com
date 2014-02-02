---
layout: post
title: "iOS と Android で画面表示時のコールバックを比較する"
date: 2014-01-29 15:55
comments: true
categories: [iOS, Android] 
---

モバイル開発における画面のライフサイクル、重要ですね。
iOS と Android で「ざっくりとは同じでしょ？」などと思っていましたが、調べてみたら結構違ってました。
<!--more-->
と言うのも、こちら

* [メモリ管理・レイアウトの観点からみた UIViewController の view の扱い - jarinosuke blog](http://blog.jarinosuke.com/entry/uiviewcontroller_view_coding_pattern)

のエントリが大変参考になったので、「Android と比べるとどうか？」と興味が沸いたのです。

## 画面が表示される時

iOS/Android の ``UIViewController``, ``Activity`` に備わってるコールバックの、画面表示時での発生順をそれぞれ調べて発生順に並べてみました。同じような意味のコールバックは横に並べて書きました。

|順番|イベント|iOS(UIViewController)|Android(Activity)|
|---|---|---|---|
|1|クラスが生成された時|init|ctor(コンストラクタ)
|2|画面がロードされる前|loadView|onCreate
|3|(画面が再度開始される前)||onRestart ※停止状態(onStop)から復帰する時のみ
|4|画面が開始される前||onStart
|5|画面がロードされた後|viewDidLoad|onPostCreate
|6|画面が表示され始める前||onResume ※一時停止(Pause)からの復帰はここから
|7|画面が表示され始めた後||onPostResume
|8|UIの配置が行われる前|viewWillLayoutSubviews|
|9|UIの配置が行われた後|viewDidLayoutSubviews|
|10|画面が表示される直前| viewWillAppear |
|11|画面が表示された直後|viewDidAppear|
|12|画面にフォーカスが移った直後||onWindowFocusChanged(true) ※表示される度に呼ばれる

### onCreate は生成前？後？

iOS というか CocoaTouch の命名文化って、will とか did とか、時系列が明確に分かるものが多いので良いですね。
それに比べて Android は…。 onCreate は前？後？ onPostCreate があるので「前」ですね。

### UIパーツのサイズはいつ決まるのか？

iOS の方は 9. ``viewDidLayoutSubviews`` の時です。
冒頭で紹介したエントリにも以下のように書かれています。

> self.view の subviews.frame の調整、すなわちレイアウト処理は全てここで記述するべきです。

Android の方は問題です。
``Button`` などの生成は ``onCreate`` で行うのが一般的ですが、この時点では、まだレイアウトされていません。なので大抵の場合 ``button1.Height = 0`` です。
では、いつのタイミングで ``button1.Height`` に適切な値が格納されるかと言うと…、 12. ``onWindowFocusChanged(true)`` まで待たないといけません。しかもこのコールバックは、Focus が変わる度に呼ばれるので、「最初の１回」だけを取得しようと思ったら別のフラグが必要になります、あーめんどい。

続きは

* [android - When Can I First Measure a View? - Stack Overflow](http://stackoverflow.com/questions/4393612/when-can-i-first-measure-a-view)

で。私は ``View.post`` する方法が一番簡単だと思いました。

### onResume/onPostResume の命名が...

「画面が表示され始める前/画面が表示され始めた後」なんて無理やりな名前を付けてしまいました。
特に ``onPostResume`` は無理がありすぎ。
名前からは ``viewDidAppear`` に相当するとも捉えられますが、まだこの時点ではレイアウトが完了していないという、中途半端なタイミングです。何のために使えば良いのでしょう？

### 回転したらどうなるの？

iOS の場合は、8.``viewWillLayoutSubviews`` からやり直しです。つまり、ここに適切に縦横対応のレイアウト処理を記述しておけば、``didRotateFromInterfaceOrientation``など、他のコールバックでの処理は通常必要ないと思います。

Android の場合は、AndroidManifest.xml への設定なしだと、なんと 1.コンストラクタ からやりなおしです。とその前に当然 ``onDestroy`` や ``OnSaveInstanceState`` が呼ばれるわけですが、、、それはまた別の機会に。

## まとめっぽいもの

iOS プログラミングでは今まで ``viewDidLoad`` で、UIパーツを生成してレイアウト処理してるプログラムが多いように思いますが、それは間違いで、「loadView で生成して、viewDidLayoutSubviews でレイアウト」とするのが最も効率的なようです。

Android でも、onCreate でレイアウト処理するとハマることがありそうです([実際ありました](http://qiita.com/amay077/items/070ac1db6b52dd03505f))。ちょっと注意しといた方がよさそうです。

「画面が破棄される時」「メモリが足りなくなった時」「回転した時」とか、書くこと沢山あるんですけど、ありすぎてもうダメです。

## 参考

### iOS

* [UIViewController Class Reference](https://developer.apple.com/library/ios/Documentation/UIKit/Reference/UIViewController_Class/Reference/Reference.html#//apple_ref/occ/instm/UIViewController)
* [メモリ管理・レイアウトの観点からみた UIViewController の view の扱い - jarinosuke blog](http://blog.jarinosuke.com/entry/uiviewcontroller_view_coding_pattern)
* [iOS View Controllerプログラミングガイド](https://gist.github.com/shinyaohira/6482235)

### Android

* [Starting an Activity | Android Developers](http://developer.android.com/training/basics/activity-lifecycle/starting.html)
* [Activity | Android Developers](http://developer.android.com/reference/android/app/Activity.html)
* [moveCamera(CameraUpdateFactory.newLatLngBounds(… で落ちる](http://qiita.com/amay077/items/070ac1db6b52dd03505f)

