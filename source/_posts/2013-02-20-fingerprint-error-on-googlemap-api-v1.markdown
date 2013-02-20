---
layout: post
title: "Google Maps API Key の取得で「入力されたフィンガープリントは無効です。」が出るようになった件"
date: 2012-11-20 18:14
comments: true
categories: [Android, GoogleMapsAPI]
---
Android で Googleマップを利用する際に必ず通る道である Google Map API Key の取得。
久しぶりに行ったら、少しハマったのでメモ。

手順は、

[Maps API Keyの取得 - Android Wiki](http://wikiwiki.jp/android/?Maps%20API%20Key%A4%CE%BC%E8%C6%C0)

など、たくさん出てくるが、この通りやっても **「入力されたフィンガープリントは無効です。」** とエラーになってしまう。

「いやいやご冗談を。」と、よぉ〜く見直したら、

##keytool の結果が MD5 じゃなくて SHA1 になってるッ！！！

という話。

```
$  keytool -list -keystore ~/.android/debug.keystore 
```

>キーストアのパスワードを入力してください:  
>
*****************  WARNING WARNING WARNING  *****************
*キーストアに保存された情報の整合性は*
*検証されていません。整合性を検証するには*
*キーストアのパスワードを入力する必要があります。*
*****************  WARNING WARNING WARNING  *****************
>
キーストアのタイプ: JKS
キーストア・プロバイダ: SUN
>
キーストアには1エントリが含まれます
>
androiddebugkey,2011/10/05, PrivateKeyEntry, 
証明書のフィンガプリント(SHA1): xx:xx:xx:xx:xx:xx:xx:…

上のリンク先のコメントにチラッと説明があった。
どうやら Java7 を導入すると SHA1 に替わってしまうらしい。

-v を足して実行すると、MD5 も表示されるので、そこからコピーして解決。

```
$  keytool -list -keystore ~/.android/debug.keystore -v
```

>キーストアのパスワードを入力してください:  
>
*****************  WARNING WARNING WARNING  *****************
*キーストアに保存された情報の整合性は*
*検証されていません。整合性を検証するには*
*キーストアのパスワードを入力する必要があります。*
*****************  WARNING WARNING WARNING  *****************
>
キーストアのタイプ: JKS
キーストア・プロバイダ: SUN
>
キーストアには1エントリが含まれます
>
別名: androiddebugkey
作成日: 2011/10/05
エントリ・タイプ: PrivateKeyEntry
証明書チェーンの長さ: 1
証明書[1]:
所有者: CN=Android Debug, O=Android, C=US
発行者: CN=Android Debug, O=Android, C=US
シリアル番号: 4e8bdcd5
有効期間の開始日: Wed Oct 05 13:28:05 JST 2011終了日: Fri Sep 27 13:28:05 JST 2041
証明書のフィンガプリント:
	 MD5:  xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:…
	 SHA1: xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:…
	 SHA256: xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:…
	 署名アルゴリズム名: SHA1withRSA
	 バージョン: 3

気づかねぇよ、こんなもん（←やつあたり