---
layout: post
title: "Mac で Windows の仮想マシンを使って IE のテスト環境を構築する"
date: 2013-07-21 15:00
comments: true
categories: [IE, HTML5]
---
Microsoft の [modern.IE](http://www.modern.ie/ja) というサイトで、Webサイトの IE対応を支援するサービスをいくつか提供していますが、その中に、「IEx 入りの仮想マシンの提供」が行われています。
<!--more-->
Virtual PC, Hyper-V, VirtualBox, VMware Fusion, Parallels などの主要な仮想環境用のイメージを配布しています。

ただし、仮想マシンの期限が 90days です。また OS の言語設定が英語です。

90days を過ぎるとまた再構築しなければならないので、日本語環境で使用するための初期設定をメモっておきます。

## 仮想マシンをダウンロードする

http://www.modern.ie/ja/virtualization-tools#downloads

から、ホストPC の OS と仮想マシンの形式を選択します。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive01.png)

次に、テストしたい IE と OS のバージョンの組み合わせから、目的の仮想マシンをダウンロードします。Mac の場合は "Grab them all with cURL" の内容を Terminal にコピペして、一括ダウンロードができます。

私は、ホストが Mac、目的は IE10+Win7 をダウンロードしました。

ダウンロードすると、 IE10.Win7.For.MacVMware.part01.sfx 〜 IE10.Win7.For.MacVMware.part04.rar というファイルができているので、これを結合するために、Terminal で、以下のコマンドを実行します。(上のサイトの 「手順」 に書いてある内容です)

```sh
chmod +x IE10.Win7.For.MacVMware.part01.sfx
./IE10.Win7.For.MacVMware.part01.sfx
```

コマンドの処理が終わると ``IE10.Win7.For.MacVMware.vmwarevm``(9.75GB) ができています。これが仮想マシンのイメージファイルです。

## 仮想マシンを起動する

ファイルを直接実行するなり、仮想マシンのマネージャ使うなりして、仮想マシンを起動します。

途中、「仮想マシンのアップグレードが必要です」と言われたので「はい」と答えました。
「この仮想マシンは移動またはコピーされた可能性があります。よく分からない場合は、[コピーしました] を選択してください。」と言われたので、よくわからないので「コピーしました」と答えました。

起動するとこんな画面になります。
ネットワークドライバなどがインストールされて、ホストPC とブリッジ接続でインターネットにもつながります。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive02.png)

## 日本語キーボードを設定する

まず、キーボードが英語なので、日本語キーボードを導入します。
デバイスマネージャを開きます。
スタートメニュー → 検索ボックスに ``devmgmt.msc`` を打ち込むのが（説明上）手っ取り早いです。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive03.png)

Keyboards - Standard PS/2 Keyboard を右クリックして Properties を表示し、Driver タブから [Update Driver] ボタンを押します。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive04.png)

"Browse my computer …" を押します。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive05.png)

"Let me pick from…" を押します。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive06.png)

Show compatible hardware のチェックを **外し** ます。
次に (Standard keyborads) から "Japanese PS/2 Keyboard (106/109 Key Ctrl + Eisuu)" を選択して Next を押します。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive07.png)

ウィザードの手順に従ってインストール完了後、仮想マシンを一旦再起動します。

**再起動後、この時点でもまだキーボード配列は英語のままです。**

次にシステムの言語設定を Japanese にします。

コントロールパネル から "Change display language" を押します。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive08.png)

"Administrative" タブから "Change system locale" を押し、"Japanese" を選択します。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive09.png)

その後、また再起動します。

再起動後、タスクトレイに [EN] という言語選択のアイコンが表れるので、[JP] にすると日本語キーボードになります。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive11.png)

いちいち切り替えるのは面倒なので、デフォルトを Japanese にしておきます。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive12.png)

ついでに、時刻や通過表示も Japanese にします。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive13.png)

忘れていました。タイムゾーンの変更もしておきます。
コントロールパネル の "Change the time zone" から、[Osaka, Sapporo, Kyoto] を選択します。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/how_to_use_ie_testdrive14.png)

## 表示を日本語にする

この Windows 7 は Enterprise Edition らしいので、

* [Windows 7 Ultimate または Windows 7 Enterprise を搭載しているコンピューターに提供される Windows 7 の言語パック](http://support.microsoft.com/kb/972813/ja)

にある方法で日本語の言語パックを導入できるかなーと思ったんですが、Windows Update で言語パックの選択が表れず、うまくいきませんでした。

どうしても日本語表示がいい！という方は、公式な方法ではないようですが、こちらの方法で日本語化できるようです。(おすすめはしないのでリンク貼りません、自己責任で。)

ttp://nagabuchi.jugem.jp/?eid=443

## Windows Update とかウイルス対策とか

テスト環境と言えど、最低限のセキュリティ対策は行なっておきましょう。
Windows Update を行なって最新の状態に、ウイルス対策は [Microsoft Security Essentials](http://windows.microsoft.com/ja-jp/windows/security-essentials-download) をインストールします。

これで最低限の環境設定ができました。
ではまた３ヶ月後にお会いしましょう。