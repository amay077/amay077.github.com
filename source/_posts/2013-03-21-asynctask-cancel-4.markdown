---
layout: post
title: "AsyncTask を cancel した時の動き【その３】"
date: 2010-11-24 7:18
comments: true
categories: [Android, Java]
---
さて前回は、AsyncTask の doInBackground 内で isCancelled をチェックして処理を中断する方法を確認しました。
<!--more-->
しかし前回の処理では、カウンタ値が 101 (期待するのは 100) になってしまいました。

今回は、AsyncTask というよりもマルチスレッド処理では必須な排他制御を行います。

またいきなり回答なんですが、前回のコードを以下のように修正します。

AsyncTask のテストプログラム３

```java
public class AsyncTestActivity extends Activity {
    private int count = 0;
    private MyTask task = null;
    private Object objLock = new Object();

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

            // 複数スレッドで同時に処理されないように保護する。
            synchronized (objLock) {
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

AsyncTestActivity で objLock を定義・生成し、doInBackground の処理を synchronized(objLock) で囲みます。

こうすることで、囲まれた内部処理は、複数スレッドから並列処理されなくなります（あとから実行されたスレッドは、synchronized の箇所で待つことになります）。

この内容は、AsyncTask というより Java (というより一般的なプログラム言語) のイロハだと思いますので、詳しい説明は、他のサイトに譲ります。

このプログラムの実行結果は次のようになります。

> taskA - cancel() が呼ばれました。
> taskB - 100 - 100030ms

今度は、taskB の結果が確実に 100 になります。synchronized によって、taskA が処理されている間(キャンセルされるまで)は taskB は待っているため、正しくカウンタが初期化・加算されます。

という事で 前々回、前回 からのまとめです。

* AsyncTask.cancel を呼んでも処理は継続中である
* 処理を中断するには、doInBackground 内で isCancelled をチェックして中断処理を行う
* マルチスレッド処理では synchronized で共有情報を保護しよう

これで AsyncTask のキャンセル処理がだいたい理解できたかなーと思います。 次は、AsyncTask.cancel() の引数「mayInterruptIfRunning」について突っ込んでみたいと思います。true と false で何か変わるの？