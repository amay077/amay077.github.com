---
layout: post
title: "Xamarin.iOS で GPS を使う"
date: 2013-03-18 00:02
comments: true
categories: [Xamarin, iOS, C#]
---
Xamarin.iOS で GPS を使ってみます。
ちなみ当方、iOS開発についてはシロートに毛が生えた程度なため、Objective-C でも GPS は使ったことありません。
<!--more-->
ので、こちらのサンプルを、Xamarin.iOS で書きなおしてみました。

* [GPSを利用する方法 - プログラミングノート](http://d.hatena.ne.jp/ntaku/20090228/1235816377)


```c# LocationSample.cs
// LocationManager 的なやつ
private CLLocationManager _man = null;

public override void ViewDidLoad()
{
    base.ViewDidLoad();

    _man = new CLLocationManager();
	
    // ボタンをタップした時
    btnListen.TouchUpInside += (s, _) => 
    {
        _man.DesiredAccuracy = 5000; // 希望精度5kmくらい
        _man.LocationsUpdated += (sender, e) => // 位置を受信した時のイベント
        {
            var l = e.Locations[e.Locations.Length - 1];
            
            lblLocation.Text = String.Format("Lat/Lng = {0}/{1}", 
                                             l.Coordinate.Latitude, l.Coordinate.Longitude);
        };
        
		// 受信開始
        _man.StartUpdatingLocation();
    };
}
```

簡単すぎるやばい。
Xamarin.iOS のクラスライブラリが CoreLocation をうまくラップしてくれて、``LocationsUpdated`` てなイベントも用意してくれてます。(Android の LocatiomManager にはイベントはなかった)
そして何度も言いますが Obj-C のキモい構文じゃないのでコードが見やすい書きやすい。

もうちょっとちゃんとしたサンプルは公式をみて下さい。

## 参考
* [GPSを利用する方法 - プログラミングノート](http://d.hatena.ne.jp/ntaku/20090228/1235816377)
* [Core Location | xamarin](http://docs.xamarin.com/samples/CoreLocation)