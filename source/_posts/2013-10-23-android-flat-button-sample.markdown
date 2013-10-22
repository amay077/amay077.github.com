---
layout: post
title: "フラットな感じのボタンを作ってみた"
date: 2013-10-23 01:53
comments: true
categories: [Android]
---
Foursquare の Android 版をマネて、XML だけで作ってみた。
仕組みをすぐ忘れるので、ここにメモしておく。
<!--more-->
```xml flat_panel.xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
    <item android:id="@+id/padding">
        <shape>
            <solid android:color="#00FFFFFF" />
            <padding android:bottom="8dp" android:left="8dp"
                android:right="8dp" android:top="8dp" />
        </shape>
    </item>
    <item android:id="@+id/shadow">
        <shape>
            <solid android:color="#D3CEC7" />
            <padding android:bottom="2px" />
        </shape>
    </item>
    <item android:id="@+id/face">
        <shape>
            <stroke android:width="1px" android:color="#AEA8A3" />
            <solid android:color="#FFFFFF" />
        </shape>
    </item>
    <item android:id="@+id/state">
        <selector>
            <item android:state_pressed="true">
                <shape android:shape="rectangle">
                    <solid android:color="#FF82DEFF" />
                </shape>
            </item>
            <item android:state_focused="true">
                <shape android:shape="rectangle" >
                    <solid android:color="#4082DEFF" />
                </shape>
            </item>
          </selector>
    </item>
    <item android:id="@+id/child">
        <shape>
            <solid android:color="#00FFFFFF" />
            <padding android:bottom="8dp" android:left="8dp"
                android:right="8dp" android:top="8dp" />
        </shape>
    </item>
</layer-list>
```

## 使い方

上のファイル ``flat_panel.xml`` を、``res/drawable`` ディレクトリに入れて、適用したい Button などに↓のように設定。

```xml part_of_activity_main.xml
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/flat_panel"
        android:text="フラットなボタン" />
```

Theme とかにまとめたいけど、今日は割愛。

## ボタンに適用した結果

### 通常

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/android_flat_button_sample_01.png)

### 押した時

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/android_flat_button_sample_02.png)

## 仕組み

``<layer-list>`` タグの中の要素は、上から順に描画される。また、下位(手前)の要素は、上位(奥)の要素の入れ子になるみたい。

![img1](https://dl.dropboxusercontent.com/u/264530/qiita/android_flat_button_sample_03.png)

（図の id は flat_panel.xml と対応してます。）

1. padding - ボタンの周りに少し余白を付ける役割（要らないなら消してもOK）
2. shadow - ボタンの下部にちょっとだけ見えてる影の部分。大部分が face によって隠れてるけど、実際はほとんど同じ領域を持つ。ここで padding bottom を 2px としているので、face の高さが 2px 縮んで、影っぽく見える仕組み。つまり影の強さはここで調整。
3. face - ボタンの「面」に該当。stroke でフチを、solid で面を塗りつぶしている。
4. state - ボタンの状態によって色などを変える役割。ここでは state_pressed=true（押された状態）と、state_focused=true（フォーカスを持ってる状態）だけ対応してる。face を描画した「後」で評価されるので、face の色を置き換えるものではない事に注意（透過時）。あと、状態は上から評価される。
5. child - LinearLayout など Group な View にこのスタイルを適用した時、子View への余白となる。

みんなの大好きな方眼紙EXCEL とオートシェイプで再現してみた → [DL](https://dl.dropboxusercontent.com/u/264530/qiita/flat_panel.xlsx)

Activity の背景が白だと、あんまり映えない。。。

色の定義を、別の xml に分けて、アプリ毎に変えれば、それっぽくなるのかなーと。