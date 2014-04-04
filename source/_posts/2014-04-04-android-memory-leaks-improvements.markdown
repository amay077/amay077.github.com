---
layout: post
title: "Activity がメモリリークしにくくなってる件"
date: 2014-03-19 15:12
comments: true
categories: [Android]
---
2009年の情報なんですけどね。
<!--more-->
* [Avoiding memory leaks | Android Developers Blog](http://android-developers.blogspot.jp/2009/01/avoiding-memory-leaks.html)
* [Avoiding memory leaks （超訳） - Android Zaurusの日記](http://d.hatena.ne.jp/androidzaurus/20090121/1232519066)
* [暇なメモ帳: Androidのソースコードレビュー(メモリリーク)](http://tomokey.blogspot.jp/2011/05/android.html)

Android でメモリリークする典型的なパターンとして上で紹介されているものがあって、日頃はこうならないように気をつけて実装をしているわけです。

また、メモリリークの調査方法もたくさん情報があります。

* [Androidでメモリリークの調査と、そのヒープダンプから画像を抽出する » RainbowDevilsLand](http://rainbowdevil.jp/?p=1187)
* [富豪的 Android プログラマの為の Eclipse Memory Analyzer Tool 入門 - sandbox](http://tlync.hateblo.jp/entry/20111220/1324372308)

日頃、Xamarin.Android を触っているので、「Xamarin でも同じようにリークするよね」と思いやってみたところ全然リークしなかったので、もしや Android-Java でもリークしないんじゃ？と考え、試してみたのが以下の内容です。

## 試した

以下の2つのパターンについて試しました

1. Avoiding memory leaks の 2番目の例。Activity への強参照を持った Drawable を static なメンバにキープしちゃう件。画面が回転した時に、Activity がリークしてしまう、とされる。
2. 暇なメモ帳さんの「問題3」＋α。非static な Inner クラスが Activity の強参照を持ってる、且つ、このオブジェクトを Activity の static メンバにしちゃう。

## 結論

から言うと、

**1. はリークせず、2. はリークしました。**

あれれ？

## パターン1のテストコード

ほぼ元コードのコピペだけど、クラスが破棄された(``finalize ``)時にログ吐くようにしています。

```java
public class MainActivity extends Activity {
    private static final String TAG = "MainActivity";
    private static Drawable sBackground;
    
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
		Log.d(TAG, "onCreate:" + this.hashCode());
		
        Button button = new Button(this);
        button.setText("Leaks are bad");
        
        if (sBackground == null) {
            sBackground = getResources().getDrawable(R.drawable.ic_launcher);
        }
        button.setBackgroundDrawable(sBackground);
        setContentView(button);
    }
	
    @Override
    protected void onDestroy() {
    	Log.d(TAG, "onDestroy:" + this.hashCode());
    	super.onDestroy();
    }
    
    @Override
    protected void finalize() throws Throwable {
    	Log.d(TAG, "finalize:" + this.hashCode());
    	super.finalize();
    }
}
```

### 確認手順

1. このアプリを実行。Android2.3 のエミュレータ(4.0 の実機でも試した)。
2. 画面を回転させる（Ctrl+F11）
3. DDMS から GC を走らせる
4. LogCat を収集

![](https://dl.dropboxusercontent.com/u/264530/qiita/improve_activity_leaks_02.png)
![](https://dl.dropboxusercontent.com/u/264530/qiita/improve_activity_leaks_03.png)

Logcat の出力結果はこう。

```
03-19 21:28:09.539: D/MainActivity(382): onCreate:1079076320
03-19 21:29:15.979: D/MainActivity(382): onDestroy:1079076320
03-19 21:29:15.989: D/MainActivity(382): onCreate:1079106528 ←横画面のActivity
03-19 21:29:33.939: D/MainActivity(382): finalize:1079076320
```

ちゃんと GC を走らせた後、 Activity の ``finalize`` が呼ばれています。
MAT でも確認したけど、リークは発見できませんでした。

## パターン2のテストコード

こんな実装は早々お目にかからないと思うけど、非static な Inner クラスのインスタンスを、Activity の static メンバにしちゃうぞ、と。

```java
public class MainActivity extends Activity {
    private static final String TAG = "MainActivity";
	private static SomeInnerClass innerClass;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.d(TAG, "onCreate");
        
        if (innerClass == null) {
            innerClass = new SomeInnerClass();
        }
    }
    
    @Override
    protected void onDestroy() {
    	Log.d(TAG, "onDestroy");
    	super.onDestroy();
    }
    
    @Override
    protected void finalize() throws Throwable {
    	Log.d(TAG, "finalize");
    	super.finalize();
    }

    class SomeInnerClass {
        public void doSomething() { }
    }
}
```

### 確認手順

パターン1と同じです。

Logcat の出力結果はこちら。

```
03-19 21:42:55.289: D/MainActivity(476): onCreate
03-19 21:43:05.369: D/MainActivity(476): onDestroy
03-19 21:43:05.549: D/MainActivity(476): onCreate
```

ご覧のとおり、``finalize`` が呼ばれない、つまり Activity がリークしています。

## 考察っぽいの

パターン2 がリークするのは当然と言えます。
Activity への強参照を持ったオブジェクトを、static フィールドで保持し続けてしまうので、Activity が破棄されない。

パターン1 も同じ理屈だと思うのですが（少なくとも冒頭の記事の説明ではそう）。これがリークしないのは、Android SDK が改善された(例えば、今まで Activity の強参照を持ってたのが弱参照に変わった)とか、Dalvik の GC が改善されたとかでしょうか？

まあ4年も経てば常識も変わるということで、Activity に関しては以前ほど神経質にならなくてもいいかもしれませんが、メモリリークの可能性が消えることは有り得ないので、このアンチパターンはこれからも遵守していかないといけませんね。


