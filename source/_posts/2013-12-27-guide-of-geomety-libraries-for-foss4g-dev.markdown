---
layout: post
title: "FOSS4G 開発者の為の、図形演算ライブラリガイド"
date: 2013-12-16 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#, Geo, FOSS4G]
---

これは [FOSS4G Advent Calendar 2013](http://atnd.org/events/45511) と、[Xamarin Advent Calendar 2013](http://qiita.com/advent-calendar/2013/xamarin)  のクロスポストになります。

地図に関するシステムを作っていますと、必ず必要になるのが図形と図形の演算です。(結合 とか、分割とか、そういうの）
<!--more-->
私にとっては自分で実装するのは、とても大変な部類なのですが、今日では、いろいろなオープンソースライブラリが存在していて、それを使わせて頂いています。

今日は、それらの紹介をします。

## 図形演算ライブラリ達

### JTS Topology Suite (JTS)

* http://sourceforge.net/projects/jts-topo-suite/
* 言語：Java
* ライセンス：LGPL

これがなければ死んでいた案件多し。いろいろな言語に移植され、事実上標準のライブラリ。LGPL ということだけが要注意であり少し残念。

### GEOS

* http://trac.osgeo.org/geos/
* 言語：C++
* ライセンス：LGPL

JTS を C++ に移植したライブラリ。なので機能、ライセンスともに JTS とほとんど一緒。osgeo のツールをビルドする時に出てくること多い。

### Net Topology Suite (NTS)

* https://code.google.com/p/nettopologysuite/
* 言語：C#
* ライセンス：LGPL

JTS を .NET に移植したライブラリ。最初見た頃は、Not Implemented  な機能が多かったけど、だいぶ揃ってきたのかな。

### Esri Geometry API for Java

* https://github.com/Esri/geometry-api-java
* 言語：Java
* ライセンス：Apache 2.0

この間の FOSS4GJ 2013 Tokyo で教えてもらった、Yet Another 図形演算ライブラリ。GIS の世界シェアトップである ESRI社がオープンソースで公開しています。だから品質は折り紙つき（のハズだ）。Apache ライセンスなのも嬉しい。

### Clipper

* http://www.angusj.com/delphi/clipper.php
* 言語：Delphi、C++、C#、Python、Perl、Ruby、Haskell
* ライセンス：Boost Software License

名前の通り Clip(つまり AND(Intersection)演算)と Offset(Buffer の片側だけ)に特化したライブラリ。

### Boost:Geometry

* http://www.boost.org/doc/libs/1_55_0/libs/geometry/doc/html/index.html
* 言語：C++
* ライセンス：Boost Software License

そういえば、C++ の拡張ライブラリである Boost にも Geometry が入ったのでしたね。Screenshot がなかなか圧巻です。

### DotSpatial

* http://dotspatial.codeplex.com/
* 言語：C#
* ライセンス：LGPL

ライブラリというよりはアプリケーションなのかな？ソースコードの中に ``DotSpatial.Topology`` などが見えます。

### JSTS Topology Suite

* https://github.com/bjornharrtell/jsts
* 言語：JavaScript
* ライセンス：LGPL

探してみたらやっぱりあった JTS の JavaScript への移植版。ライセンスは(ry

### GeoScript

* http://geoscript.org/
* 言語：JavaScript
* ライセンス：MIT

JavaScript製のライブラリ。最近は D3.js による視覚表現が流行ってきたので、内部ではこのようなライブラリが使われているのでしょうか。

## 試しに使ってみよう

Xamarin Advent Calendar と絡めるために無理やり Xamarin Studio で、という事は必然的に Net Topology Suite を使ってみます。

Xamarin Studio は、Android/iOS アプリを作るためだけじゃなくて、コンソールアプリとかも作ることができますよ、と言いたいだけです。

### 準備

まず Xamarin Studio で C# → コンソールアプリのプロジェクトを作ります。

次に、まず NTS を参照に追加しますが、Nuget という仕組みを使います。
Xamarin Studio に Nuget を導入する手順は、

* [mrward / monodevelop-nuget-addin](https://github.com/mrward/monodevelop-nuget-addin)

を参考にしてください。

### コードを書く

こんな感じです。

```csharp Program.cs
using System;
using GeoAPI.Geometries;
using NetTopologySuite;

namespace TopologyTest
{
    class MainClass
    {
        public static void Main(string[] args)
        {
            var service = NtsGeometryServices.Instance;
            var gf = service.CreateGeometryFactory();

            var polygonA = gf.CreatePolygon(new Coordinate[]
                {
                    new Coordinate(34.0, 136.0),
                    new Coordinate(34.0, 138.0),
                    new Coordinate(37.0, 138.0),
                    new Coordinate(37.0, 136.0),
                    new Coordinate(34.0, 136.0)
                });

            var polygonB = gf.CreatePolygon(new Coordinate[]
                {
                    new Coordinate(36.0, 137.0),
                    new Coordinate(35.0, 137.0),
                    new Coordinate(35.0, 140.0),
                    new Coordinate(36.0, 137.0)
                });

            polygonA.Intersection(polygonB).ToConsole("Intersection");
            polygonA.Union(polygonB).ToConsole("Union");
            polygonA.SymmetricDifference(polygonB).ToConsole("SymmetricDifference");
            polygonA.Difference(polygonB).ToConsole("Difference");
            polygonB.Buffer(0.5).ToConsole("Buffer");
        }

    }

    public static class GeomExtensions
    {
        public static void ToConsole(this IGeometry geom, string tag) {
            Console.WriteLine(tag + " - " + geom.ToString());
        }
    }
}
```

Intersection(AND)、Union(OR)、SymmeticDifference(XOR)、Difference(A - B)、Buffer(ふくらます)について試しています。

実行すると、コンソールに結果の座標群がずらーと出力されます。

## 見える化してよ

プログラムによる視覚化は、、、ごめんなさい面倒だったので作りませんでした。

その代わり、GeoJSON 化して GitHub にアップして視覚化しました。

まず演算対象の ``geometryA`` と ``geometryB`` です。(外側の枠は気にしないでください)

* https://github.com/amay077/geojsontest/blob/master/01_polygonA.geojson

<script src="https://embed.github.com/view/geojson/amay077/geojsontest/master/01_polygonA.geojson"></script>

* https://github.com/amay077/geojsontest/blob/master/02_polygonB.geojson

<script src="https://embed.github.com/view/geojson/amay077/geojsontest/master/02_polygonB.geojson"></script>

### Intersection(AND)

* https://github.com/amay077/geojsontest/blob/master/03_polygonA_intersection_B.geojson

<script src="https://embed.github.com/view/geojson/amay077/geojsontest/master/03_polygonA_intersection_B.geojson"></script>

### Union(OR)

* https://github.com/amay077/geojsontest/blob/master/04_polygonA_union_B.geojson

<script src="https://embed.github.com/view/geojson/amay077/geojsontest/master/04_polygonA_union_B.geojson"></script>

### SymmetricDifference(XOR)

* https://github.com/amay077/geojsontest/blob/master/05_polygonA_symmetricdifference_B.geojson

<script src="https://embed.github.com/view/geojson/amay077/geojsontest/master/05_polygonA_symmetricdifference_B.geojson"></script>

### Difference(A - B)

* https://github.com/amay077/geojsontest/blob/master/06_polygonA_difference_B.geojson

<script src="https://embed.github.com/view/geojson/amay077/geojsontest/master/06_polygonA_difference_B.geojson"></script>

### Buffer(Bを膨らます)

* https://github.com/amay077/geojsontest/blob/master/07_geometryB_buffer.geojson

<script src="https://embed.github.com/view/geojson/amay077/geojsontest/master/07_geometryB_buffer.geojson"></script>


いやー 便利ですね GitHub 。大量のマーカーは自動的にクラスター化までしてくれるそうですよ。

* [GeoJSON rendering improvements](https://github.com/blog/1541-geojson-rendering-improvements)。

こちらは、 Leaflet.js、OpenStreetMap、[Maki Project](https://www.mapbox.com/maki/) などの FOSS4G が使われています。いいですね。

さて、なんの話か分からなくなってきたので、こちらからは以上です。