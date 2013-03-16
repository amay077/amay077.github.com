---
layout: post
title: "Xamarin.Android で GPS を使う(Reactive Extensions版)"
date: 2013-03-17 00:42
comments: true
categories: [Xamarin, Android, C#, ReactiveExtensions]
---
RxM4A により [Reactive Extensions が使えるようになった](http://amay077.github.com/blog/2013/03/01/how-to-use-rx-in-xamarin/) ので、以前に [Android+reactive4Java でやったコレ](http://amay077.github.com/blog/2012/10/03/locate-using-reactive4java/) を Xamarin.Android でやってみます。
<!--more-->
```c# LocationManager.Extenstion.cs
namespace Amay077.Android.Locations
{
    static class LocationManagerExtenstion
    {
        // 渡した Action<Location> を OnLocationChanged で実行されるようにしただけ
        class LocationListener : Java.Lang.Object, ILocationListener
        {
            private readonly Action<Location> _locationChangedHandler;
            
            public LocationListener(Action<Location> locationChangedHandler)
            {
                _locationChangedHandler = locationChangedHandler;
            }
            
            #region ILocationListener implementation
            public void OnLocationChanged(Location location)
            {
                _locationChangedHandler(location);
            }
            
            public void OnProviderDisabled(string provider) { }
            public void OnProviderEnabled(string provider) { }
            public void OnStatusChanged(string provider, Availability status, Bundle extras) { }
            #endregion
        }

        public static IConnectableObservable<Location> RequestLocationAsObservable(
            this LocationManager locMan,
            string provider)
        {
            return Observable.Create<Location>(o => 
            {
                try {
                    var isStop = false; // RemoveUpdates してもすぐ止まるか分からんので一応フラグ持っとく

                    var listener = new LocationListener(l => 
                    {
                        if (isStop) return;
                        o.OnNext(l);
                    });

                    // 位置取得開始
                    locMan.RequestLocationUpdates(provider, 0, 0, listener);
                    
                    return () => // Dispose() した時に停止
                    {
                        if (isStop) return;
                        isStop = true;
                        locMan.RemoveUpdates(listener);
                        o.OnCompleted();
                    };

                } catch (Exception ex) {
                    o.OnError(ex);
                    return () => { /* empty */ };
                }
            }).Publish(); // Hot な Observable に
        }
    }
}
```

``LocationManager`` の拡張メソッドにしたいので、クラス名を慣例に習って ``LocationManager.Extenstion.cs`` に、メソッド ``RequestLocationAsObservable`` の第一引数に ``this`` を付けてます。

``LocationListener`` Inner クラスは、C# では匿名クラスが使えないので、コンストラクタで指定した ``Action`` が ``OnLocationChanged`` で呼ばれるようにしただけです。

``RequestLocationAsObservable`` メソッドがメイン。やってることは reactive4Java と同じです。Hot な Observable にしたので、最後に ``.Publish()`` してるので、返値が ``IConnectableObservable`` になってます。

さて使う方。

```c# Howtouse.cs
using Amay077.Android.Locations; // 拡張メソッドを使えるように

<省略>

// LocationManager を得る
var locationMan = (LocationManager)context.GetSystemService(Context.LocationService);
// RequestLocationAsObservable があたかも LocationManager のメンバのよ(ry
var observable = locationMan.RequestLocationAsObservable(LocationManager.GpsProvider);
observable.Take(3) // 3回取得
.Timeout(new TimeSpan(10000)) // 10秒待って取得できなかったらタイムアウト
.Subscribe(
    l => { Android.Util.Log.Debug(TAG, 
              String.Format("received: {0}/{1}", 
                l.Latitude, l.Longitude)); }, // 位置が取得される度に呼ばれる
    e => Android.Util.Log.Debug(TAG, "error:" + e.Message), // エラー(タイムアウト含む)の時呼ばれる
    () => Android.Util.Log.Debug(TAG, "finished.")); // 全部終わったら呼ばれる

observable.Connect(); // 接続開始
```

Reactive Extensions と C# の拡張メソッドなどのおかげで、Java よりもずいぶんとすっきり書けました。
