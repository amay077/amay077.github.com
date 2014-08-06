---
layout: post
title: "Android の static 変数はヤバい"
date: 2011-01-21 7:36
comments: true
categories: [Android, Java]
---
今まで Windows アプリケーションしか作ったことのない人（私のような）が、Android アプリケーションをつくりはじめて戸惑うところの一つに、Activity のライフサイクルがあります。今回これ関連で、 見事に static 変数でハマりましたので、メモしておきます。
<!--more-->

Android Developper では、Activity のライフサイクルは以下の図で説明されています。

!["image1"](http://developer.android.com/images/activity_lifecycle.png)

この図の説明は、[日本Androidの会のページ](http://www.android-group.jp/index.php?cmd=read&page=%CA%D9%B6%AF%B2%F1%2FAndroid%20SDK%20WG%20%C2%E81%B2%F3%20%A5%BB%A5%C3%A5%B7%A5%E7%A5%F3%A1%CA2008.10.25%A1%CB&word=activity#iee83184) などで解説されているので、ここでは割愛しますが、ポイントなのは、

アプリケーションを終了（onDestroy）しても、プロセスは死んでいない。
という事です。static 変数でハマったと書きましたが、static 変数はプロセス毎に保持されるものですが、Windows アプリケーションでは、アプリケーションを終了して再起動すれば、static 変数も初期化されます。

Android アプリケーションでも普通そうだろう、と思うのが人情です。

で、上のライフサイクルです。この図では、アプリケーションを終了してもプロセスが終了するとは書かれていません。他のアプリケーションによってメモリが必要になった時だけ、プロセスが殺される、とあります。これはつまり、Android アプリケーションでは、起動→終了→再起動しても、（ほとんどの場合）static 変数は初期化されない、という事になります。

ホントにそうでしょうか？サンプルを作ってためしてみました。

### static 変数の生存を確認するサンプル

{% gist 789458 %}

static 変数を用意し、LogCat に出力した後、1加算するだけです。上記のコードを実行すると、LogCat にカウンタ値が出力されます。↓こんな感じで。

> 01-21 18:16:54.937: DEBUG/MainActivity(2869): static member:0

さて、これを「戻る」で終了し、再び起動してみます。

> 01-21 18:17:43.443: DEBUG/MainActivity(2869): static member:1

カウンタが加算されている事が分かります。なんども終了→起動を繰り返すとどんどん加算されていきます。

> 01-21 18:17:43.443: DEBUG/MainActivity(2869): static member:2
> 01-21 18:17:45.112: DEBUG/MainActivity(2869): static member:3
> 01-21 18:17:48.876: DEBUG/MainActivity(2869): static member:4 

このようにアプリケーションを終了しても static 変数は（プロセスは）生存していることが分かりました。

次に破棄された事を確認してみます。他のアプリでメモリが必要になった時～というのは再現させにくいので、タスク管理ソフトを使います。

自分は、[SystemPanel Lite](http://jp.androlib.com/android.application.nextapp-systempanel-iFtq.aspx) を使いました。このソフトで作成したアプリケーションを選択してタスクを終了させた後、再度アプリケーションを起動してみます。すると、

> 01-21 18:17:50.223: DEBUG/MainActivity(2869): static member:0

出力結果はこのようになり、タスクが終了させられたので、カウンタも 0 に戻った事が確認できます。

このように、Android アプリケーション開発では、static 変数は要注意です。

各Activity から共有できるからと多用すると、特に状態を格納するなどで使うと不具合の元になりそうです。

※だれか Activity じゃなくて ”アプリケーション" が終了するタイミングを補足する方法、教えてください

<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=oku2008-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B00KNOQ8UE" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>

<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=oku2008-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B00KX5FKCA" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
