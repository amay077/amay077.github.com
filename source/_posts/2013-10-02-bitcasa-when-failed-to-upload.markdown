---
layout: post
title: "Bitcasa でアップロードに失敗したファイルが見つかった時の対処方法"
date: 2013-10-02 19:30
comments: true
categories: [Bitcasa]
---
相変わらず「安定している」とは言いがたい Bitcasa さん。CPU 喰いまくったり、アップロード残が一向に減らないまま１日経過とかザラで、そういう時はアプリを終了させたり、プロセスを殺したりする訳ですが、そうしますと「正常にアップロードできてないファイル」ができる事があります。
<!--more-->
このファイルは、Bitcasa の Web サイトで「灰色」で表示されます(下を参照)。このファイルは Web 上で再生も、ダウンロードもできません。一方、PC の Finder や Explorer 上の Infinite Drive では、特に問題ないように見えます。が、再生や実行などしようとするとエラーになります。(キャッシュに残っている内はうまくいくかも)

![img](https://support.bitcasa.com/attachments/token/yw6awz1fvdktbuf/?name=noname.png)

この症状について、[Bitcasa のサポートサイト](https://support.bitcasa.com/) で、以下の情報が見つかります。

* [some pictures can not be opened on my.bitcasa.com : Help Center](https://support.bitcasa.com/entries/23768267-some-pictures-can-not-be-opened-on-my-bitcasa-com)

これ、要約すると、

「キャッシュを消してアップロードしなおしてください」

と言ってるんですが、次の理由でやってられません。

**「どんだけフォルダあると思っとんねん！」**

そう、「アップロードに失敗したファイルを見つける方法」は、Web サイトで「灰色になっているのを目視する」しかないのです。

他のユーザーもこれを問題視していて、[Bitcasa の要望受付](http://feedback.bitcasa.com/forums/184524-bitcasa-feature-requests-suggestions) には、次のような案が挙げられています。


1. [A incomplete files list in my bitcasa portal](http://feedback.bitcasa.com/forums/184524-bitcasa-feature-requests-suggestions/suggestions/3982870-a-incomplete-files-list-in-my-bitcasa-portal)
2. [Mark incomplete/corrupted files with a modified date of 01-01-1970 so that file sync utilities will sync them again](http://feedback.bitcasa.com/forums/184524-bitcasa-feature-requests-suggestions/suggestions/4403970-mark-incomplete-corrupted-files-with-a-modified-da)
3. [A option to delete incomplete files from my Bitcasa Drive](http://feedback.bitcasa.com/forums/184524-bitcasa-feature-requests-suggestions/suggestions/4115823-a-option-to-delete-incomplete-files-from-my-bitcas)


1 が一番投票数が多くて実現の検討に入ってる(IN REVIEW)ようです。

で、タイトルにある「対処方法」としては、 **「Vote してください」** となります。なんじゃそら。

しかし、クラウドストレージのキモって「確実に」「クラウド上に」「ファイルが保管されている」ことだと思うんですが、この辺が危ういのなんとかしてほしいですねえ。