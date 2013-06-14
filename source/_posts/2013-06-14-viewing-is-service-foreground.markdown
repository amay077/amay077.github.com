---
layout: post
title: "Service が Foreground で動いているか確認する"
date: 2013-06-14 18:11
comments: true
categories: [Android]
---
Android で「死ににくいサービス」を作るには [Service.startForegound](http://developer.android.com/reference/android/app/Service.html#startForeground(int, android.app.Notification) を呼び出す必要がありますが、動いてるサービスがちゃんと「Foregound になってるか？」は以下のようにして確認できます。(Notification 表示が強制されるから通知バー見ればいいじゃん、と言われればそうなんですけど、ちゃんとしたエビデンスっぽいのが欲しくて)
<!--more-->
## 手順

```sh
adb shell dumpsys activity s <サービス名>
```

<サービス名> はサービスの完全名称を入れます。AndroidManifest.xml の ``<service android:name=`` で定義したやつ。

実行すると以下のように出力されます。

> ACTIVITY MANAGER SERVICES (dumpsys activity services)
>   User 0 active services:
>   * ServiceRecord{41b2dd18 u0 com.amay077.android.gpsfaker/.service.GpsSignalService}
>     intent={cmp=com.amay077.android.gpsfaker/.service.GpsSignalService}
>     packageName=com.amay077.android.gpsfaker
>     processName=com.amay077.android.gpsfaker
>     baseDir=/data/app/com.amay077.android.gpsfaker-1.apk
>     dataDir=/data/data/com.amay077.android.gpsfaker
>     app=ProcessRecord{416e19e8 3209:com.amay077.android.gpsfaker/u0a10072}
>     isForeground=true foregroundId=2130968576 foregroundNoti=Notification(pri=0 contentView=com.amay077.android.
> (以下省略)

``isForegound=true`` と表示されているので、確かに「このサービスはフォアグラウンドだ」と分かります。サービス側で ``startForeground`` を呼び出さなかった場合は、この項目は表れません。

``dumpsys`` 今までコマンドめんどいと思ってあまり使ってませんでしたすいませんでした(汗

## 参考

大変参考になりました。

* [Yukiの枝折: Android:Service.dumpでサービスの状態をダンプする](http://yuki312.blogspot.com/2013/02/androidservicedump.html)