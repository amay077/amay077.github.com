---
layout: post
title: "TextView で、テキストの一部だけクリック可能にする"
date: 2013-10-17 21:16
comments: true
categories: [Android]
---
Android の TextView で、「テキストの一部を押すとなんかのアクションが起こる」というのが ``ClickableSpan`` というのを使えばできそうだったので、やってみました。
<!--more-->
```java ClikcableSpanTestActivity.java
public class ClikcableSpanTestActivity extends Activity {
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        final List<Pair<String, String>> data = new ArrayList<Pair<String,String>>();
        data.add(new Pair<String, String>("りんご", "隠し味としてカレーに入れます"));
        data.add(new Pair<String, String>("みかん", "あぶりだしに使います"));
        data.add(new Pair<String, String>("すいか", "種を食べると盲腸になります（嘘）"));
        
        final SpannableStringBuilder sb = new SpannableStringBuilder();
        final String SEP = ", ";
        int spanStart = 0;

        for (final Pair<String, String> p : data) {
            sb.append(p.first);
            sb.append(SEP);

            // 追加した文字列を Span にする
            sb.setSpan(new ClickableSpan() {
				@Override
				public void onClick(View widget) {
					Toast.makeText(getApplicationContext(), 
						p.second, Toast.LENGTH_SHORT).show();
				}
			}, spanStart, spanStart + p.first.length(), Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
            
            spanStart += (p.first.length() + SEP.length());
		}
        
        TextView tv = (TextView)findViewById(R.id.text);
        tv.setText(sb);
        tv.setMovementMethod(LinkMovementMethod.getInstance()); // これ忘れるとクリックできなくて小一時間悩む
    }
}
```

項目をカンマつなぎで表示して、項目のところだけリンクっぽくします。
リンクをクリックすると Toast を表示します。（下図のように）
また ``URLSpan`` というクラスもあり、クリックすると指定したURLに移動（ブラウザアプリが起動）します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_clickablespan_01.png)

## 参考

装飾したり位置やサイズ変えたり、TextView でもいろいろできるんですねー。
(VB6の)Label とは違うのだよ、Label とは！

* [Y.A.M の 雑記帳: Android Spannable を使って文字列の一部を装飾する](http://y-anz-m.blogspot.com/2011/08/androidspannable.html)
* [Tips TextView を使いこなそう ～表示編～ その4 - - Google Android - 雑記帳](http://d.hatena.ne.jp/androidprogram/20100529/1275086958)
* [android.text.style | Android Developers](http://developer.android.com/reference/android/text/style/package-summary.html)