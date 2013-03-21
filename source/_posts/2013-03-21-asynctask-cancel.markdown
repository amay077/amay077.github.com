---
layout: post
title: "AsyncTask を cancel した時の動き【その２】"
date: 2010-11-22 7:05
comments: true
categories: [Android, Java]
---
さて 前回 は、タスク実行中に AsyncTask.cancel を呼んでも、処理自体は中断されない事を確認しました。
<!--more-->

今回をそれを中断させる方法を確認していきます。

いきなり答えなんですが、処理を中断させるには、doInBackgroud の要所要所で、AsyncTask.isCancelled() をチェックして、 true なら処理を中断するコードを自力でコーディングします。

AsyncTask のテストプログラム２

```java
public class AsyncTestActivity extends Activity {
    private int count = 0;
    private MyTask task = null;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        Button btn = (Button)findViewById(R.id.Button01);
        btn.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View view) {

                // タスク A を実行
                task = new MyTask("taskA");
                task.execute((Void)null);

                // ５秒待つ
                SystemClock.sleep(5000);

                // タスク A を中断
                task.cancel(true);

                // タスク B を実行
                task = new MyTask("taskB");
                task.execute((Void)null);
            }
        });
    }

    /** 非同期で加算を行う内部クラス */
    class MyTask extends AsyncTask {
        private String name = "";

        MyTask(String name) {
            this.name = name;
        }

        /** 非同期処理 */
        @Override
        protected Long doInBackground(Void... params) {
            long start = System.currentTimeMillis();

            // count を 0～100 まで 100ms 毎に加算する処理
            count = 0;
            for (int i = 0; i < 100; i++) {
                // キャンセルされたら抜ける
                if (this.isCancelled()) {
                    return 0;
                }

                SystemClock.sleep(100);
                count++;
            }

            // 処理にかかった時間を返す
            return System.currentTimeMillis() - start;
        }

        /** doInBackground が終わったら呼び出される。 */
        @Override
        protected void onPostExecute(Long result) {
            // 結果を表示 "タスク名 - カウンタ値 - 処理時間ms"
            Toast.makeText(AsyncTestActivity.this,
                name + " - " + String.valueOf(count) + " - " 
                            + String.valueOf(result) + "ms",
                Toast.LENGTH_LONG).show();
        }


        /** cancel() がコールされると呼び出される。 */
        @Override
        protected void onCancelled() {
            // 結果を表示 "タスク名 - cancel() が呼ばれました。"
            Toast.makeText(AsyncTestActivity.this,
                name + " - cancel() が呼ばれました。",
                Toast.LENGTH_LONG).show();
        }
    }
}
```

doInBackground のループ内で isCancelled が true だったら抜ける、という処理を追加しています。

また、cancel を呼んだらメッセージが表示されるように onCancelled にメッセージ処理を追加しています。

このプログラムの実行結果は次のようになります。

> taskA - cancel() が呼ばれました。
> taskB - 101 - 100029ms

taskA で cancel を実行しているので、まず1行目が出力されます。

ここでの注意点は、onCancelled が呼び出されるのは処理が中断された時ではなく、cancel が呼び出された後、だという事です。

また、前回の出力でも分かる事ですが、cancel を呼んだ時は、onPostExecute は呼び出されません。 なので、例えば onPreExecute でプログレスダイアログを表示している場合は、キャンセルを考慮するなら、onPostExecute だけでなく onCancelled でもダイアログを閉じる処理をしなければなりません。

2行目の出力ですが、前回は 151 までカウンタが加算されてしまった事に比べればマシになりましたが、まだ完全ではないです。 count は タスク A と B で共有しているので、確実に排他をかける必要があるでしょう。

では、今回のまとめです。

* cancel を実行すると、AsyncTask.isCancelled() が true を返す。
* doInBackground で this.isCancelled をチェックして true になったら中止する処理を実装する。
* でも排他処理は必要

という事で[次回](/blog/2010/11/24/asynctask-cancel-4/)は、排他制御について書きます。