---
layout: post
title: "moveCamera(CameraUpdateFactory.newLatLngBounds(… で落ちる"
date: 2013-09-29 21:16
comments: true
categories: [Android, Java, GoogleMapsAPI]
---
Google Map Android API v2 では、指定した範囲にいいかんじにズームしてくれるメソッドがあって（これを使うと下記事のようなことができる）、とても便利なのですが、普通に使ってたら落ちました（泣
<!--more-->
* [Android GooglMapを使い、現在値と目的地を（２点間）を表示させる。 | App Camp](http://tryworks-design.com/?p=1530)

その理由と、対策を記録しておきます。

## エラーになるコード

GoogleMap v2 を使ったよくあるコード。

```java MainActivity.java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    SupportMapFragment fragment = ((SupportMapFragment)getSupportFragmentManager()
            .findFragmentById(R.id.map));
    GoogleMap gmap = fragment.getMap();
    
    LatLngBounds bounds = LatLngBounds.builder()
        .include(new LatLng(35.4433011,139.646108)) // 横浜
        .include(new LatLng(35.6846001,139.696919)) // 東京
        .build();
    
    gmap.moveCamera(CameraUpdateFactory.newLatLngBounds(bounds, 15));
}
```

起動時に、横浜-東京 が画面内に入るようにズームする、つもりのコード。

これは以下のエラーになる。

>09-29 20:22:58.508: E/AndroidRuntime(18904): FATAL EXCEPTION: main<br/>
09-29 20:22:58.508: E/AndroidRuntime(18904): java.lang.RuntimeException: Unable to start activity ComponentInfo{com.amay077.<br/>android/com.amay077.android.mapsample.view.MainActivity}: java.lang.IllegalStateException: Map size should not be 0. Most likely, layout has not yet occured for the map view.

## エラーの原因

StackOverflow さまに載ってた。

* [android - moveCamera with CameraUpdateFactory.newLatLngBounds crashes - Stack Overflow](http://stackoverflow.com/questions/13692579/movecamera-with-cameraupdatefactory-newlatlngbounds-crashes)

また、[APIリファレンス](https://developers.google.com/maps/documentation/android/views#changing_camera_position) にも次のように記載がある。

>Note: Only use the simpler method newLatLngBounds(boundary, padding) to generate a CameraUpdate if it is going to be used to move the camera after the map has undergone layout. During layout, the API calculates the display boundaries of the map which are needed to correctly project the bounding box. In comparison, you can use the CameraUpdate returned by the more complex method newLatLngBounds(boundary, width, height, padding) at any time, even before the map has undergone layout, because the API calculates the display boundaries from the arguments that you pass.

意訳すると ``newLatLngBounds(boundary, padding)`` は、レイアウトが完了した後で使ってね、そうでない場合は、``newLatLngBounds(boundary, width, height, padding)`` を使ってね。ということらしい。

確かに ``onCreate`` ではまだレイアウトされていないので納得。

## 対策

上の StackOverflow でも解決策として、``ViewTreeObserver.addOnGlobalLayoutListener`` を使って、レイアウトが完了したタイミングで moveCamera する方法が紹介されているが、もうちっとシンプルにできないかなと思っていたところ、ちょうど別件で「ビューのサイズが確定(して Width/Height が取得できる)タイミング」を調べていて、同じく StackOverflow で以下の情報を発見。

* [android - When Can I First Measure a View? - Stack Overflow](http://stackoverflow.com/questions/4393612/when-can-i-first-measure-a-view/15301092#15301092)

これによると ``view.post(new Runnable() { … })`` のタイミングでも OK らしいので、今回はこれを使ってみる。

```java MainActivity.java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    SupportMapFragment fragment = ((SupportMapFragment)getSupportFragmentManager()
            .findFragmentById(R.id.map));
    final GoogleMap gmap = fragment.getMap();
    
    // NOTE MainActivity.this.runOnUiThread(new Runnable() { ではダメだった
    fragment.getView().post(new Runnable() {
        @Override
        public void run() {
            LatLngBounds bounds = LatLngBounds.builder()
                    .include(new LatLng(35.4433011,139.646108)) // 横浜
                    .include(new LatLng(35.6846001,139.696919)) // 東京
                    .build();
            
            gmap.moveCamera(CameraUpdateFactory.newLatLngBounds(bounds, 15));
        }
    });
}
```

としたところ、正常に地図がズームされました。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/movecamera_with_cameraupdatefactory_newlatlngbounds_crashes_01.png)

ちなみに、処理をメインスレッド上で行う ``Activity.runOnUiThread`` や ``Handler.post`` では NG、冒頭と同じエラーでした。処理は UIスレッド上で行われるけど、Map はまだレイアウト未完了、という事だと思います。

起動時の処理は、全ての View で post 内に書いておいた方がいいのかも。
