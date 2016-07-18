---
layout: post
title: "ボタンをタップした時に○○する、を Java と Swift と Xamarin で比較する"
date: 2016-07-15  23:59:59 +0900
comments: true
categories: [Android, Xamarin, Swift, C#, iOS]
---

たぶん一番書くやつを 

<!--more-->

* Android-Java
* Android-Xamarin
* iOS-Swift
* iOS-Xamarin 

で比較。

----
## Android-Java

```java MainActivity.java
button1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        label1.setText("pushed!!");        
    }
});
```
----
## Xamarin.Android(C#)

```csharp MainActivity.cs
buttonOk.Click += (_, e)
    => label1.Text = "pushed!!";
```
----
## iOS-Swift

```java ViewController.swift
buttonOK.addTarget(self, action: 
    #selector(ViewController.onTouch(_:)), 
    forControlEvents: .TouchUpInside)
・・・
func onTouch(sender: AnyObject) {
    label1.text = "pushed!!"
}
```
----
## Xamarin.iOS(C#)

```csharp ViewColtroller.cs
buttonOk.TouchUpInside += (_, e) 
    => label1.Text = "pushed!!";
```
----
## まとめ

**Xamarin はいいぞ！**

