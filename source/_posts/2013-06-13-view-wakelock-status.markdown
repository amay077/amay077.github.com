---
layout: post
title: "WakeLock の状態を確認する"
date: 2013-06-13 20:58
comments: true
categories: [Android]
---
どのアプリが WakeLock を取得しているかを見る方法。
<!--more-->
```sh
adb shell dumpsys power
```

を実行して、ずらっと出力される中から "Wake Locks:" を探す。
出力されるのはこんな感じの情報。

>
>POWER MANAGER (dumpsys power)
>
>Power Manager State:
>  mDirty=0x0
>  (中略)
>
>Settings and Configuration:
>  mDreamsSupportedConfig=true
>  (中略)
>
>Screen off timeout: 30000 ms
>Screen dim duration: 6000 ms
>
>Wake Locks: size=1
>  SCREEN_BRIGHT_WAKE_LOCK        'WindowManager' ON_AFTER_RELEASE (uid=1000, pid=389, ws={WorkSource: uids=[10070]})
>
>(以下省略)


"Wake Locks: size=1" となっており、
'WindowManager' という TAG で ``SCREEN_BRIGHT_WAKE_LOCK`` が取得されているのが分かる。

## 参考

* [adb各種コマンド - 肉になるメモ](http://kazumeat.hatenablog.com/entry/20110814/1313295257)