---
layout: post
title: "Xamarin.iOS と Rx で iBeacon を使ってみた"
date: 2013-12-11 00:12
comments: true
categories: [Xamarin, iBeacon, iOS, C#, XAC13]
---
こちらは、[iBeacon Advent Calendar 2013](http://qiita.com/advent-calendar/2013/ibeacon) と [Xamarin Advent Calendar 2013](http://qiita.com/advent-calendar/2013/xamarin) とのクロスポストになります。

Xamarin とは、.NET で iOS/Android アプリを開発できるプラットフォームです。詳しくは [こちら](http://qiita.com/amay077/items/38ee79b3e3e88cf751b9) をどうぞ。

Xamarin.iOS は、 iOS の APIセットが全て C# で使えますので、 iBeacon 関連の API もそのまま使えます。さらに C# や .NET の強力な言語仕様により、より簡潔に、美しく書くことができます。

## Xamarin.iOS で iBeacon を使うサンプル

Xamarin で iBeacons を使うサンプルは、Xamarin 自体が既に公開しています。

* [Play ‘Find The Monkey’ with iOS 7 iBeacons | Xamarin Blog](http://blog.xamarin.com/play-find-the-monkey-with-ios-7-ibeacons/)
* [mikebluestein/FindTheMonkey](https://github.com/mikebluestein/FindTheMonkey)

これらは iOS7 のリリースから僅か7日後のできごとであり、Xamarin の新OSへの対応力に驚いたものでした。

この紹介だけで終わってもアレなので、このサンプルをより「C# っぽく」修正してみたいと思います。

## サンプルをより「C# っぽく」する

対象にするのは iBeacon の受信の方です。

* [iBeaconの解説 - Reinforce-Lab.'s Blog](http://reinforce-lab.github.com/blog/2013/10/21/ibeacon/)
* [iBeacon Tips: 正しいビーコン監視方法 | ブライテクノBlog](http://brightechno.com/blog/archives/220)

などで勉強したところ、受信の流れは下図のようになるかと思います。

![img](https://dl.dropboxusercontent.com/u/264530/qiita/using_ibeacon_with_xamarin_01.png)

全てのメソッドが非同期でコールバックを受け取るタイプ、また並行処理＆同期とか、なんだか見やすいコードになる気がしません。

C# といえば LINQ、そして LINQ を更に拡張する Rx(Reactive Extensions) を使って、この流れをもう少しスッキリと書いてみます。
Rx は、非同期処理やイベントコールバックを一直線なストリームに変換します。また、ストリームの分配や結合の機能を提供します。最初は JavaScript の [Deferred](http://techblog.yahoo.co.jp/programming/jquery-deferred/) みたいなもんだと思ってました。が、使ってく内にとんでもなく高機能なものだと分かり(はじめ)ました。

これを使うと、上の図をそのままコードに落としたような、上から下へ辿れる感じで書くことができます。

まず、修正前のサンプルコードはこちら

* [FindTheMonkey / FindTheMonkeyViewController.cs](https://github.com/mikebluestein/FindTheMonkey/blob/master/FindTheMonkey/FindTheMonkeyViewController.cs)

このコード自体、上の流れに沿ってない気もしますが、まあいいや。

これを Rx で書きなおすと、こうなります。

### 2014.4.2 追記

実際には動かない空想のコードを掲載していたので、実機で動作したコードに書き換えました。

```csharp FindTheMonkeyViewController_after.cs
if (!UserInterfaceIdiomIsPhone)
{
  /* 省略 */
} else
{
  InitPitchAndVolume();
  var man = new CLLocationManager();

  man.StartMonitoringAsObservable(beaconRegion) // 監視開始
  .SelectMany(r =>
    man.RegionEnteredAsObservable() // A:進入の受信
    .Amb(man.RequestStateAsObservable(r) // B:リージョン状態要求
      .Where(e => e.State == CLRegionState.Inside) // 範囲内のみ
      .Select(x => x.Region as CLBeaconRegion) // CLRegion からcast
    .SelectMany(man.StartRangingBeaconsAsObservable) // A/B どちらかを受信したらレンジング開始
      .Where(x => x.Beacons.Length > 0) // ビーコンが1個以上みつかった場合のみ
      .Select(x => x.Beacons [0]) // LINQ の Fisrt() でもOk
      .DistinctUntilChanged(x => x.Proximity) // Proximity が変わった時のみ流す
  .Subscribe((CLBeacon beacon) =>
  {
    // Beacon が見つかった時に行う処理を書く
  });
}
```

```csharp CLLocationManagerExtensions.cs
public static class CLLocationManagerExtensions
{
  // リージョン監視を開始して、開始通知を IObservable で得る拡張メソッド
  public static IObservable<CLBeaconRegion> StartMonitoringAsObservable(
    this CLLocationManager man, CLBeaconRegion beaconRegion)
  {
    return Observable.Defer(() =>
    {
      var o = Observable.FromEventPattern<CLRegionEventArgs>(
            h => man.DidStartMonitoringForRegion += h, 
            h => man.DidStartMonitoringForRegion -= h)
      .Select(x => x.EventArgs.Region as CLBeaconRegion);
      
      man.StartMonitoring(beaconRegion);          
      return o;
    });
  }
  
  // リージョンへの進入を IObservable で得る拡張メソッド
  public static IObservable<CLBeaconRegion> RegionEnteredAsObservable(
    this CLLocationManager man)
  {
    return Observable.FromEventPattern<CLRegionEventArgs>(
      h => man.RegionEntered += h, h => man.RegionEntered -= h)
        .Select(e => e.EventArgs.Region as CLBeaconRegion);
  }
  
  // リージョンの状態を要求して、結果を IObservable で得る拡張メソッド
  public static IObservable<CLRegionStateDeterminedEventArgs> RequestStateAsObservable(
    this CLLocationManager man, CLBeaconRegion beaconRegion)
  {
    return Observable.Defer<CLRegionStateDeterminedEventArgs>(() => 
    {
      var o = Observable.FromEventPattern<CLRegionStateDeterminedEventArgs>(
      h => man.DidDetermineState += h, h => man.DidDetermineState -= h)
        .Select(e => e.EventArgs);
      
      man.RequestState(beaconRegion);
      return o;
    });
  }
  
  // レンジングを開始してビーコン信号を IObservable で得る拡張メソッド
  public static IObservable<CLRegionBeaconsRangedEventArgs> StartRangingBeaconsAsObservable(
    this CLLocationManager man, CLBeaconRegion beaconRegion)
  {
    return Observable.Defer(() => 
    {
      var o = Observable.FromEventPattern<CLRegionBeaconsRangedEventArgs>(
      h => man.DidRangeBeacons += h, h => man.DidRangeBeacons -= h)
        .Select(e => e.EventArgs);
      
      man.StartRangingBeacons(beaconRegion);
      return o;
    });
  }
}
```

実機で動作するサンプルを

* [amay077/FindTheMonkey](https://github.com/amay077/FindTheMonkey)

に置きました。

``locationMgr.StartMonitoringAsObservable`` で始まるところがキモですね。各々の機能は ``CLLocationManagerExtensions.cs`` の拡張メソッドで逃してます。これも C# の利点(たしか Objective-C にもあったっけ)。

リージョン監視の開始通知を受け取ったら進入の検知(A)をしつつ、もうひとつの処理で開始位置のリージョン状態を得て、それがリージョン内だったら値を流す(B)。これらは ``.Amb`` で合流。 ``.Amb`` は右辺と左辺のどちらか先に返された最初の結果を後続に流すというものです。つまり、B がリージョン外だったら自動的に A の ``didEnterRegion`` を待つことになります。

最後に、レンジングを開始して受信する度に結果(ビーコン信号)を流します。
 
んで、これを購読(``.Subscribe``)することで処理を開始して、結果を ``// Beacon が見つかった時に行う処理を書く`` のところで受け取る仕組みです。

このように Rx を使うことで、非同期のコールバックを含む処理を直列に書け、処理の並列化や合成も簡単に行えます。

Objective-C でも [ReactiveCocoa](http://qiita.com/somtd@github/items/8409ddd6d0927c04c1dd) とか使うとできるのかな？(でもやっぱり構文が…)

そんなわけで、少しでも Xamarin に興味持っていただけたら幸いです。(これが言いたかった)