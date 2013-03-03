---
layout: post
title: "Xamarin.Android でオレオレ Application クラスを使う"
date: 2013-03-03 17:48
comments: true
categories: [Xamarin, Android, C#]
---
オレオレApplicationクラスって、[コレ](http://techbooster.org/android/application/2353/)のことなんですが、正式名称知らないので勝手にこう呼んでます(^_^;)
<!-- more -->
Xamarin.Android では、Application クラスを継承するのに加えてもう二手間くらい必要みたいです。割と苦労したのでメモしておきます。

## AndroidManifest.xml へ追記
Xamarin.Android では最初は AndroidManifest.xml は存在しないのですが、[このような手順](http://amay077.github.com/blog/2013/03/02/xamarin-android-permission/)で追加できます。

んで、\<application> タグに ``android:name`` 属性を追記します。ここは Android本家と同じ要領です。

```xml AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" 
    android:versionCode="1" android:versionName="1.0" package="HelloXamarinAndroiid.HelloXamarinAndroiid">
	<uses-sdk />
	<application android:label="HelloXamarinAndroiid" android:name=".MyApplication">
	</application>
</manifest>
```

## Application クラスを継承したクラスを作る

次に、クラスの実装ですが、Application クラスから派生させる他、

* [Application] 属性を付ける
* ``(IntPtr, JniHandleOwnership)`` なコンストラクタを用意する

ことが必要なようです。

こんなかんじ。

```c# MyApplication.cs
namespace HelloXamarinAndroiid
{
    [Application] // この属性が必要らしい
    class MyApplication : Application
    {
        // このコンストラクタを明示的に override 剃る必要があるらしい
        public MyApplication (IntPtr javaReference, JniHandleOwnership transfer)
            : base(javaReference, transfer)
        {
        }

        public override void OnCreate()
        {
            base.OnCreate();
            // Test
            Android.Util.Log.Debug("MyApplication", "OnCreate called.");
        }
    }
}
```

アプリを動かすと、ログが出力されてるのでこの手順でいいのかと。

## 参考

* [Working with AndroidManifest.xml | xamarin](http://docs.xamarin.com/guides/android/advanced_topics/working_with_androidmanifest.xml) 公式。ちょっとこれだけじゃ分かんなかったっす。
* [c# - Custom Application child class in Mono for Android - Stack Overflow](http://stackoverflow.com/questions/9928386/custom-application-child-class-in-mono-for-android) Application そして StackOverflow のお世話に。
