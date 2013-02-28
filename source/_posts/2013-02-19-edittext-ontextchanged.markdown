---
layout: post
title: "Android/EditTextでIMEの未確定文字列が確定された瞬間(のフォーク)"
date: 2012-12-26 0:00
comments: true
categories: [Android]
---
Android の EditText の文字列と、String 変数値を同期させたいケースって結構あると思うんですけど、[LostFocus](http://developer.android.com/reference/android/view/View.OnFocusChangeListener.html) みたいなのでやると、オンフォーカスな状態で Activity が閉じられた時に LostFocus が呼ばれなくてあぼんとなるのは VB6あがりのプログラマ(=私)なら誰もが経験するんじゃないでしょうか？

<!-- more -->

じゃあ [onTextChanged](http://developer.android.com/reference/android/text/TextWatcher.html#onTextChanged(java.lang.CharSequence, int, int, int) だぜ、って仕込んでみると、Android のこれは IME で変換中の文字列もバンバン飛んで来まして大変使い勝手が悪い。(サジェストなんかする際には必要なんでしょうけども)

例えば、以下で紹介されている方法

* [TextWatcherでEditTextの入力内容をリアルタイムに反映する | GE Android Blog](http://blog.global-eng.co.jp/android/2011/04/08/textwatcher%e3%81%a7edittext%e3%81%ae%e5%85%a5%e5%8a%9b%e5%86%85%e5%ae%b9%e3%82%92%e3%83%aa%e3%82%a2%e3%83%ab%e3%82%bf%e3%82%a4%e3%83%a0%e3%81%ab%e5%8f%8d%e6%98%a0%e3%81%99%e3%82%8b/)

を実装しますと、IME確定前の文字列もじゃんじゃん同期してくれちゃいます。

これに起因したであろう Android アプリもありまして(Instagram とか。今は治ってる。)、なんとかならんかなーと思っていました。

そんな時、こちら↓

* [Android/EditTextでIMEの未確定文字列が確定された瞬間 | SpiriteK Blog](http://www.spiritek.co.jp/spkblog/2012/10/25/androidedittext%e3%81%a7ime%e3%81%ae%e6%9c%aa%e7%a2%ba%e5%ae%9a%e6%96%87%e5%ad%97%e5%88%97%e3%81%8c%e7%a2%ba%e5%ae%9a%e3%81%95%e3%82%8c%e3%81%9f%e7%9e%ac%e9%96%93/)

の記事にめぐり逢いまして、まさに私が求めていたそのもの。

ですが、未確定文字＝下線付き(``UnderlineSpan``)である、という前提がどうにもしっくり来ませんで(未確定文字に下線を付けない IME もそりゃあるだろうなーという意味で)。

もちろん、その前で言及されている内部クラス ``android.view.inputmethod.ComposingText`` を文字列比較するのもうーん…。

で、自分でも試行錯誤してみたところ、[Spanned.getSpanFlags](http://developer.android.com/reference/android/text/Spanned.html#getSpanFlags(java.lang.Object) というメソッドがあるのに気づきました。あと、[SPAN_COMPOSING](http://developer.android.com/reference/android/text/Spanned.html#SPAN_COMPOSING) というフラグも。

これらを使って、文字列が確定されているかどうか？を識別することができるのではないかと考え、前出の記事のコードを以下のように修正してみました。

```java DetermineComposingText.java
edit1.addTextChangedListener(new TextWatcher()
{
    int currentLength = 0;

    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {}

    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {
        currentLength = s.toString().length();
    }

    @Override
    public void afterTextChanged(Editable s) {
        Log.v("", "after:" + s.toString());
        if (s.toString().length() < currentLength) {
            return;
        }
        boolean unfixed = false;
        Object[] spanned = s.getSpans(0, s.length(), Object.class);
        if (spanned != null) {
            for (Object obj : spanned) {
                // UnderlineSpan での判定から getSpanFlags への判定に変更。
                // if (obj instanceof android.text.style.UnderlineSpan) {
                if ((s.getSpanFlags(obj) & Spanned.SPAN_COMPOSING) == Spanned.SPAN_COMPOSING) {
                    unfixed = true;
                }
            }
        }
        if (!unfixed) {
            Toast toast = Toast.makeText(getApplicationContext(), "確定", Toast.LENGTH_SHORT);
            toast.show();
        }
    }
});
```

``s.getSpanFlags(obj) & Spanned.SPAN_COMPOSING) == Spanned.SPAN_COMPOSING)`` で SPAN_COMPOSING(未確定文字)のフラグが立っているかを判定しています。

とりあえず手持ちの ATOK では修正前と同じように動作しているようです。

IME 側で「未確定文字は SPAN_COMPOSING を必ず設定する」ものかどうかは分かりませんが、個人的には UnderlineSpan を使った手法よりもすっきりしました、というお話でした。


