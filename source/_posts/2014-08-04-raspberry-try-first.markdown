---
layout: post
title: "いまさら Raspberry Pi やってみた"
date: 2014-08-04 21:03:00 +0900
comments: true
categories: [RaspberryPi]
---
昨年の秋に買って以来放置してしまってた Raspberry Pi、ふと思い立って動かしてみました。
<!--more-->
当時は、買ったはよかったものの、余ってるSDカードがなくてそのまま「寝て」しまってました。

## OS のインストール

* [RaspberryPi - Raspberry Pi に入門してみた。 - Qiita](http://qiita.com/tomk79/items/cdc1b88358afba2c6337)

に沿ってやりました。ZIPファイルのダウンロードが300MBほど経過したところで終わってしまったので、BitTorrent を使ってみました。
Mac なので、[Transmission](https://www.transmissionbt.com/) というクライアントを使用しました。速いっすね BitTorrent。

RaspberryPiへの給電は MacMini のUSBポートから、ディスプレイはHDMIで行いました。
SDカードは、以前使っていたスマホに刺さっていたmicroSDカード＋SDカードアダプタにて、そのためか、少し衝撃を与えるとOSが再起動してしまうという gkbr な環境です。

## WiFi に接続する

有線LANが余ってなかったので、WiFiでやることにしました。ちょうど使わなくなったノートPCにWiFiドングルが刺さってました。

WiFi に接続するための設定は、

* [コチョナナバ: Raspberry Piを無線LAN対応させてみた](http://kingyo-bachi.blogspot.jp/2013/07/raspberry-pilan.html)

を参考にしましたが、設定後再起動してもIPアドレスが取得できず、結局 ``startx`` で GUI を立ち上げ、GUIのWiFi設定ツールで設定したところ、接続できるようになりました。その後の再起動でも自動的にWiFiに接続できるようになりました。

Raspberry Pi のUSBポートは２つしかないのに、GUIでのWiFi設定を行うにはマウスとキーボードとWiFiドングルが必要で、キーボードとマウスを抜き差ししながら、しかも本体にできるだけ衝撃を与えないようにするのが大変でした（^_^;）

GUIのWiFi設定ツールで設定した後の ``wpa_supplicant.conf`` はこんな感じになっていました。

```json wpa_supplicant.conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
	ssid="<WiFi-APのSSID>"
	psk="<WiFi-APの接続生パスワード>"
	proto=RSN
	key_mgmt=WPA-PSK
	pairwise=TKIP
	auth_alg=OPEN
}
```

## root のパスワードを設定

``sudo passwd root`` で root のパスワードを設定しました。（こんなことすらググらないと分からない程度のLinux力ですorz）

## SSHで接続

前出のコチョバナナさんの続きで、 ``avahi-deamon`` を導入して、DHCPでIPアドレスが変わっても、``ssh raspberrypi.local`` で接続できるようにしました。

とりあえずここまで。
カメラとかセンサーはないので、これから何しようか考えます。

一連のセットアップで一番苦労したのは vi のテキスト入力です。。。

<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=oku2008-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B00CBWMXVE" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>


<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=oku2008-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B003NSAMW2" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>

<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=oku2008-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B00ISP0MOI" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
