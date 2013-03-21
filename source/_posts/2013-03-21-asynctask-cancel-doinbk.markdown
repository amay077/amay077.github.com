---
layout: post
title: "AsyncTask を cancel した時の動き【その１】"
date: 2010-11-17 2:43
comments: true
categories: [Android, Java]
---
Android 開発でユーザービリティを向上させるのに良く利用する AsyncTask ですが、cancel した時の内部の動作が不明だったので調べまてみました。 知りたいのは、 「cancel を呼び出したら、doInBackgroud で行われている処理はどうなるのか？」 です。
<!--more-->
そこで用意したのが下のプログラムです。 これはクラス変数 count を MyTask によって 100 まで加算します。 MyTask のインスタンスを二つ用意し、

1. タスクAを開始
2. ５秒待つ
3. タスクAをキャンセル
4. タスクBを開始

という処理をしています。 タスク A と B で count は共有しています。

AsyncTask のテストプログラム１

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
    }
}

まず前提として、タスクを１つだけ実行した時は、こんな結果になります。

###タスク A だけを実行した時の結果

>taskA - 100 - 100029ms

100ミリ秒毎に100回加算しているので、妥当な結果といえます。

このコードを実行すると次のような結果になりました。

#### テストプログラム１の結果

>taskB - 151 - 100031ms

taskA の結果は出力されないので cancel した場合は onPostExecute は呼び出されない事が分かります。

しかし、カウント値が 151 と異常になってます。 どうやら taskA と taskB が並列処理されてしまっているようです。 なので、cancel を呼び出しても、doInBackground の処理は継続して行われている事が分かります。

今回分かったことは

* AsyncTask.cancel を呼ぶと、onPostExecute は呼び出されない。
* AsyncTask.cancel を呼んだだけでは doInBackground の処理は中断されない。
です。

じゃあ、どうやって中断させようか、という事で次回↓へ続く。

AsyncTask を cancel した時の動き【その２】