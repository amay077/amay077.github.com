---
layout: post
title: "非同期処理の書き方を Java と Swift と Xamarin で比較する"
date: 2016-07-19  23:59:59 +0900
comments: true
categories: [Android, Xamarin, Swift, C#, iOS]
---

「重たい処理を非同期で実行して、結果をメインスレッドで画面に表示」を、

<!--more-->

* Android-Java
* iOS-Swift
* Xamarin(Android も iOS も同じ)

で比較。

----

## Android

```java
@Override
public void onClick(View view) {
    new AsyncTask<Void, Void, Long>() {
        @Override
        protected Long doInBackground(Void[] p) {
            // ワーカースレッド
            long ret = 0;
            for (long i = 0; i < 1000000000; i++)
                ret += i;
            return ret;
        }

        @Override
        protected void onPostExecute(Long result) {
            // UIスレッド
            text1.setText(String.valueOf(result));
        }
    }.execute((Void)null);
}
```

----

## Swift

```swift
@IBAction func onTouchUpInside(sender: AnyObject) {
    weak var weakSelf = self
    dispatch_async(dispatch_get_global_queue(
        DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), {
        // ワーカースレッド
        var ret:Int = 0
        for i in 0...1000000000 {
            ret += i
        }
        
        dispatch_async(dispatch_get_main_queue(), {
            // UIスレッド
            weakSelf?.label1.text = String(ret)
        });
    });
}
```

----
## Xamarin(Android も iOS も)

```csharp
Task<long> FatProc() => Task.Run<long>(() => {
    long ret = 0;
    for (long i = 0; i < 1000000000; i++)
        ret += i;
    return ret;
});

button1.TouchUpInside += async (_, e) => {
    var ret = await FatProc(); // ワーカースレッド
    label1.Text = ret.ToString(); // UIスレッド
};
```

----

## Xamarin はいいぞ！
