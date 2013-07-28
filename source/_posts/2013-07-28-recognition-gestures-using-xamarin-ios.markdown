---
layout: post
title: "Xamarin.iOS でジェスチャを認識する"
date: 2013-07-28 13:49
comments: true
categories: [Xamarin, iOS]
---
[Xamarin.iOS](http://xamarin.com/) でも ``UIGestureRecognizer`` が普通に使えるわけですが、Objective-C に比べてコードが短く書けて感動した話です。
<!--more-->
本日の Obj-C の先生はこちら。

* [Gesture Recognizers 〜簡単にタッチ操作を検知 | iPad Techfirm Lab ](http://labs.techfirm.co.jp/ipad/cho/466)

このサンプルを Xamarin.iOS に移植してみます。

## サンプルコード

```cs:GesturesSample_ViewDidLoad.cs
// Tap gesture
this.View.AddGestureRecognizer(new UITapGestureRecognizer(tap => 
{
    Debug.WriteLine("Double Tap.");
}) 
{ 
    NumberOfTapsRequired = 2 // Double tap 
});

// Drag(Pan) gesture
this.View.AddGestureRecognizer(new UIPanGestureRecognizer(pan => 
{
    var p = pan.TranslationInView(this.View);
    var v = pan.VelocityInView(this.View);
    Debug.WriteLine("Pan. transration:{0}, velocity:{1}", p, v);
}));

// Pinch gesture
this.View.AddGestureRecognizer(new UIPinchGestureRecognizer(pin => 
{
    var scale = pin.Scale;
    var v = pin.Velocity;
    Debug.WriteLine("Pinch. scale:{0}, velocity:{1}", scale, v);
}));

// Swipe gesture
this.View.AddGestureRecognizer(new UISwipeGestureRecognizer(sw => 
{
    Debug.WriteLine("Swipe.");
}));

// Rotate gesture
this.View.AddGestureRecognizer(new UIRotationGestureRecognizer(ro => 
{
    var rotation = ro.Rotation;
    var v = ro.Velocity;
    Debug.WriteLine("Rotate. rotation:{0}, velocity:{1}", rotation, v);
}));

// Long press gesture
this.View.AddGestureRecognizer(new UILongPressGestureRecognizer(lp => 
{
    Debug.WriteLine("Long press.");
}));
```

ViewController 全体のソースは [コチラ](https://gist.github.com/amay077/6094422)

元のサイトのサンプルコードは 70行弱ありますが、Xamarin.iOS では 45行くらいで書けました。しかも、GestureRecongnizer の登録とハンドラが同じ場所に書けるので見やすい。

しかしこれ、ハンドラとか GesutureRecognizer、破棄しなくていいのかなあ。。。