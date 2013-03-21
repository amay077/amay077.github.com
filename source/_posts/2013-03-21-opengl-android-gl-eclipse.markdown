---
layout: post
title: "OpenGLチュートリアル android-gl を Eclipse でビルドする方法"
date: 2010-12-6 1:50
comments: true
categories: [Android, Java, OpenGL]
---
[@npaka](http://twitter.com/#!/npaka) さんの [Androidメモ - OpenGLによる3Dグラフィックス](http://www.saturn.dti.ne.jp/~npaka/android/OpenGL/index.html) で解説されている OpenGLチュートリアル [android-gl](http://code.google.com/p/android-gl/) が、Eclipse でビルドできない！（たぶん Ant でビルドするサンプル）
<!--more-->

ので、色々調べてビルドできるようになったので、その手順を書いておきます。

1. SVN からソースコードをエクスポートします。
2. 他の Android プロジェクト（なんでもいい）から "default.properties" というファイルをコピーします。
3. "/src/edu/union/" ディレクトリから R.java ファイルを削除します。（R.java は本来自動生成されるものなので）
4. "/res/layout/main.xml" ファイルを <TextView id="@+id/text" から <TextView android:id="@+id/text" に修正します。 
5. Eclipse でプロジェクトをインポートします。
6. ビルドすると "android requires .class compatibility set to 5.0" というエラーが出るので、このエラーメッセージでググって解決します。（よくあるエラーみたいです）
7. もう一度ビルドすると成功するハズです。

↓こんなサンプルアプリが実行できるハズですよ、と。

!["1"](https://dl.dropbox.com/u/264530/qiita/open_gl_1.png)
!["2"](https://dl.dropbox.com/u/264530/qiita/open_gl_2.png)
!["3"](https://dl.dropbox.com/u/264530/qiita/open_gl_3.png)