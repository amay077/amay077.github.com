---
layout: post
title: "Android NDK のセットアップ方法(バージョン r4b, 2010/11/30時点)"
date: 2010-11-30 2:37
comments: true
categories: [Android, NDK]
---
Cygwin も NDK もド素人ですが、何とかできました。
<!--more-->
NDK 1.5 の時から変わっていて苦労しましたが、解説してくれているサイトやツイート、SDKドキュメントを読んで何とかできましたので、その手順を記録しました。

## Cygwin のインストール
[AndroidのNDK 1.5でHelloJNIを動かす手順 – Android(アンドロイド)情報-ブリリアントサービス](http://d.hatena.ne.jp/bs-android/20090707/1246952991#) を参考にして進めました。

ダウンロードした Cygwin は 1.7.7 でした。

途中、

> さらにスクロールし、gccのパッケージを選択(※赤丸のしるしの部分をクリック)します。自動的にgcc-coreなどにもチェックが入りますが、そのままにしておきます。

とありましたが、勝手にチェックが入らなかったので、手動で同じようにチェックを入れました。

## NDK のインストール
引き続き [AndroidのNDK 1.5でHelloJNIを動かす手順 – Android(アンドロイド)情報-ブリリアントサービス](http://d.hatena.ne.jp/bs-android/20090707/1246952991#) を参考にしました。

android-ndk-r4b-windows.zip を

> c:\cygwin\home\h_oku

に解凍しましたが、

> \android-ndk-r4b-windows\android-ndk-r4b

となってしまって混乱したので、

> c:\cygwin\home\h_oku\android-ndk-r4b

となるようにしました。

次に、

> $ sh build/host-setup.sh

としてもコマンドが無い（確かにファイルを検索しても host-setup.sh がない）ので焦りましたが、

> @saltpp (Tomoki Shiono)
>
> build/host-setup.sh が無くなって、アプリのトップディレクトリで、ndk-build コマンドを実行すれば、ビルドできるようになってるね。>NDKr4
>
> 5月21日 TweetDeckから

とのだったので、

> $ sh ndk-build

を実行しました。

…がエラーが出ました。

> $ sh ndk-build
> Android NDK: Could not find application project directory !
> Android NDK: Please define the NDK_PROJECT_PATH variable to point to it.
> /home/h_oku/android-ndk-r4b/build/core/build-local.mk:85: *** 
> Android NDK: Aborting    .  Stop.

NDK_PROJECT_PATH を定義しろ、といってるようだったので、 HOME と同じように環境変数に 「/home/h_oku/project」 と定義しました。

## サンプルを NDK でビルド
[Android NDK | Android Developers](http://developer.android.com/sdk/ndk/index.html) の

Getting Started with the NDK

に

> 1. Place your native sources under /jni/…
> 2. Create /jni/Android.mk to describe your native sources to the NDK build system
> 3. Optional: Create /jni/Application.mk.

とあったので、

> C:\cygwin\home\h_oku\android-ndk-r4b\samples\hello-jni

のフォルダとファイルを

> C:\cygwin\home\h_okuyama\project

にコピーしました。

次に、

> Build your native code by running the ‘ndk-build’ script from your projet’s directory. It is located in the top-level NDK directory: $ cd $ /ndk-build

とあったので、

> C:\cygwin\home\h_oku\project

に CD で移動し、

> C:\cygwin\home\h_oku\android-ndk-r4b\ndk-build

を実行しました。怒られました。

> C:/cygwin/home/h_oku/android-ndk-r4b/ndk-build

でうまくいきました（\ と / が違ってました）。

こんな感じの出力がされました。

> $ C:/cygwin/home/h_oku/android-ndk-r4b/ndk-build
> Compile thumb  : hello-jni <= /home/h_oku/project/jni/hello-jni.c
> SharedLibrary  : libhello-jni.so
> Install        : libhello-jni.so => /home/h_oku/project/libs/armeabi
そして、おお！

> C:\cygwin\home\h_oku\project\libs

とか

> C:\cygwin\home\h_oku\project\obj

ができている！

## Eclipse でビルド。そしてアプリの実行
再び [AndroidのNDK 1.5でHelloJNIを動かす手順 – Android(アンドロイド)情報-ブリリアントサービス](http://d.hatena.ne.jp/bs-android/20090707/1246952991#) に戻り、 Eclipse で Android Project を作成し、既存の project フォルダを指定しました。

プロジェクト名やビルド・ターゲットが自動で設定されましたが、Android 2.2 がチェックされ、上部に

> The API level for the selected SDK target does not match the Min SDK Version.

と警告が出ています。とりあえず無視して完了しました。

プロジェクトをビルド→実行し、Android 2.2 のエミュレータ上で動作を確認することができました。

NDK 側のソースを修正したら、Cygwin で、

> CD C:\cygwin\home\h_oku\project
> C:/cygwin/home/h_oku/android-ndk-r4b/ndk-build

として、C:\cygwin\home\h_oku\project\libs\armeabi\libhello-jni.so を再作成し、Eclipse 側で F5 を押して読み込みなおして実行しなおせば反映されます。

※ちなみに全角文字を表示しようとしたら落ちました…orz