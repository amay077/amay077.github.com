---
layout: post
title: "ダイアログを二重に開かないようにする"
date: 2010-11-14 8:37
comments: true
categories: [Android, Java]
---
Android なのか Java なのかどっちの仕様か知らないですが、どうやらこちらの世界には「モーダルダイアログ」という概念がないみたいで。
<!--more-->
なので以下のようなコードを書くと、連続でボタンをタップすると、ダイアログが二重三重に表示されてしまいます。

ダイアログが二重に開いちゃうコード

```java
Button button = (Button)findViewById(R.id.Button1);
button.setOnClickListener(new View.OnClickListener() {
    public void onClick(View v) {
        // 再現しやすいように少し負荷をかける
        SystemClock.sleep(1000);

        AlertDialog dlg = new AlertDialog.Builder(v.getContext()).create();
        dlg.setTitle("ほげ");
        dlg.setMessage("ほげほげ");
        dlg.setButton("閉じる", new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialog, int which) {
                dialog.dismiss();
            }
        });
        dlg.show();
    }
});
```

で自分が考えた対策がこれ。

二重に開かないコード

```java
Button button = (Button)findViewById(R.id.Button1);
stopButton.setOnClickListener(new View.OnClickListener() {
    // メンバとして宣言
    AlertDialog dlg = new AlertDialog.Builder(LoggingActivity.this).create();

    public void onClick(View v) {
        // 表示されてたら何もしない
        if (dlg.isShowing()) {
            return;
        }

        // 再現しやすいように少し負荷をかける
        SystemClock.sleep(1000);

        dlg.setTitle("ほげ");
        dlg.setMessage("ほげほげ");
        dlg.setButton("閉じる", new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialog, int which) {
                dialog.dismiss();
            }
        });
        dlg.show();
    }
});
```

匿名クラスではコンストラクタが使えないので、dlg.setTitle とか何回もよばれちゃうのが気に入らないけど、まあ目的は達成できたのでこれでいいかなと。 C# なら 2〜3行で書けるのに〜。