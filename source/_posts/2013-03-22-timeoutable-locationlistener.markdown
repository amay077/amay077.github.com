---
layout: post
title: "TimeoutableLocationListener というのを作ってみた"
date: 2011-01-13 4:30
comments: true
categories: [Android, Java, Gps]
---
Android で GPS の位置を受信する場合、LocationListener というインターフェースを渡して位置が受信できるのをジッと待つわけです。Android での GPS の使い方はググるといっぱいでてきますもんね。
<!--more-->

で、LocationManager.onStatusChanged というメソッドがあって、例えば屋内とかで、受信できずにあきらめた場合、こいつが呼び出されるのかと思いきや、期待したようなタイミングで受信してくれません。

結局自力でタイムアウト処理を書いて GPS を止めるハメになるのですが、いつも同じ処理を書くのが面倒なので、共通っぽいクラスにしてみました。

{% gist 777790 %}

LocationListener の代わりにこの TimeoutableLocationListener をセットします。いつもと違うのは以下の2箇所です。

* コンストラクタで、LocationManager , タイムアウト時間（ミリ秒）, タイムアウト時に呼び出されるリスナ を渡す。
* 自分のクラスで onLocationChanged を Override した時に、必ず base.onLocationChanged を呼び出す。

interface じゃなくなっちゃったので、ちょっと不自由ですが、まあ使えるのではないかと思います。