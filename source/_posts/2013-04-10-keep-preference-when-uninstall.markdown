---
layout: post
title: "SharedPreference は消さずにアプリだけアンインストールする方法"
date: 2013-04-10 19:23
comments: true
categories: [Android]
---
ADB のコマンドで、

```sh
adb shell pm uninstall -k <package name>
```
<!--more-->
を実行します。

ヘルプには、

>pm uninstall: removes a package from the system.
>Options:
>    -k: keep the data and cache directories around after package removal.

と書いてあります。

「おっしゃ、別なPCでビルドしてインストールしたアプリの設定情報消したくなかったんだよね〜」

と思いきや、署名(debug.keystore とか)が異なる環境でビルドされたアプリの場合、上記のコマンドでアンインストールしても、新しいアプリをインストールする時に、

>[2013-04-10 17:58:29 - MyApp] Installation error: INSTALL_FAILED_UPDATE_INCOMPATIBLE
[2013-04-10 17:58:29 - MyApp] Please check logcat output for more details.
[2013-04-10 17:58:29 - MyApp] Launch canceled!

というエラーが出てアンインストールできません。
結局、

```sh
adb shell pm uninstall <package name>
```

で、アンインストールすることになります。

という本末転倒なお話でした。