---
layout: post
title: "Xamarin Studio でコンポーネントを更新する方法"
date: 2013-05-08 20:49
comments: true
categories: [Xamarin, GoogleMapsAPI, iOS]
---
分かりにくかったのでメモ。
<!--more-->
[以前](http://amay077.github.io/blog/2013/04/22/xamarin-ios-using-gmap-ios-sdk/)使ってみた [Google Maps Component](http://components.xamarin.com/view/googlemapsios/)、「Polygon や Circle がないなー、対応してないのかなー」と思って [API Doc](http://componentsapi.xamarin.com/?link=T%3aGoogle.Maps.Circle) 見たら存在してたので、いつの間にかバージョンアップしてたらしい、確かに手持ちのバージョンは「1.1.2」、Webサイトの方は「1.2.2」になってた。


## Component を更新する

じゃあ更新するか、と思って Xamarin Studio で入り口を探すものの見つからない。
結局、メニュー -> Get More Components から、Google Maps を検索しなおしたら、ボタンが「Update」になってたので、押したら更新された。

![image1](https://dl.dropboxusercontent.com/u/264530/qiita/update_components_using_xamarin_studio1.png)

## Component に付属のサンプルが増えてた

更新後、Samples を見てみたら、、、お、 **iOS Advanced Sample** というのが増えてる！きっとアドバンスドなサンプルなのでしょうな（試せよ

![image1](https://dl.dropboxusercontent.com/u/264530/qiita/update_components_using_xamarin_studio2.png)

## Component を更新したら API の互換性が無くなってた

更新後、意気揚々と以前作ったサンプルをビルドしてみたらビルドエラーが。
どうやら ``MapView.AddMarker`` や ``MarkerOption`` が無くなって、``Marker`` に MapView を設定するように変更されたらしい。(以下は、さっきのアドバンスドなサンプルからの抜粋)

```c# PartOfMarkersViewController.cs
public override void ViewDidLoad ()
{
	base.ViewDidLoad ();

	var camera = CameraPosition.FromCamera (-37.81969, 144.966085, 4);
	var mapView = MapView.FromCamera (RectangleF.Empty, camera);

	var sydneyMarker = new Marker () {
		Title = "Sydney",
		Snippet = "Population: 4,605,992",
		Position = new CLLocationCoordinate2D (-33.8683, 151.2086),
		Map = mapView
	};

	var melbourneMarker = new Marker () {
		Title = "Melbourne",
		Snippet = "Population: 4,169,103",
		Position = new CLLocationCoordinate2D (-37.81969, 144.966085),
		Map = mapView
	};

	// Set the marker in Sydney to be selected
	mapView.SelectedMarker = sydneyMarker;

	View = mapView;
}
```

これは、元の Google Maps SDK for iOS が根源なのか、Google Maps の Xamarin Component がそうしたのか知りませんが、ReleaseNotes とかないのかな？うっかり更新すると怖いな。

## まとめ
* Component の Update は、 Get More Components から
* むやみに Update すると互換性なくなってるかもなのでバックアップというかバージョン管理ちゃんとしよう
* つまり Xamarin Components に ReleaseNotes 欲しいです