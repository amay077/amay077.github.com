---
layout: post
title: "Xamarin Studio で MvvmCross を使うための準備"
date: 2013-07-22 20:46
comments: true
categories: [Xamarin, MvvmCross, C#]
---
クロスプラットフォーム MVVM フレームワーク「[MvvmCross](https://github.com/slodge/MvvmCross)」を Mac の Xamarin Studio で使うための準備についてです。
<!--more-->
基本は、

* [monotouch - Getting PCL, Mvvmcross, Nuget and Xamarin Studio to play "nice" on Mac - Stack Overflow](http://stackoverflow.com/questions/17653208/getting-pcl-mvvmcross-nuget-and-xamarin-studio-to-play-nice-on-mac)

のトレース。主には PCL と Nuget が正しく動くようにする手順です。

## 前提条件

必要な(というか試した)環境は以下の通り。
（Xamarin の PCL サポートがまだ「進行中」なので、将来的には変わる可能性大）

* Xamarin Studio Version 4.0.10 (beta)
* Xamarin.Android Version: 4.8.0 (beta)
* Xamarin.iOS Version: 6.3.8.11 (beta)
* Xamarin Studio Add-in NuGet Package Management Version 0.5
* Mac OS X 10.8.4

## 手順

### Xamarin Studio

1. Mac に Xamarin Studio を入れて、Beta チャンネルに切り替えて更新。
2. アドインマネージャから NuGet Package Management をインストール。

### .NETPortable DLLs を Win機からコピってくる

Win機の ``C:¥Program Files (x86)¥Reference Assemblies¥MicrosoftFramework.NETPortable`` を、Mac の ``/Library/Frameworks/Mono.framework/External/xbuild-frameworks/.NETPortable/`` へコピー。 

Win機がない場合は、[これ](http://amay077.github.io/blog/2013/07/21/building-testing-environment-for-mac-using-ietestdrive/) などで Win仮想環境を作り、[Visual Studio Ultimate 2012 90日間試用版](http://www.microsoft.com/visualstudio/jpn/downloads) を入れるとよい(Express 版は上記DLLsがないのでNG)。

### Nuget にパッチをあててビルド

[ここ](http://stackoverflow.com/questions/17653208/getting-pcl-mvvmcross-nuget-and-xamarin-studio-to-play-nice-on-mac) の **Patch to Nuget.Core.dll:** にあるテキストを適当なファイルに保存(ここでは ``patch.diff`` とする)して、以下のコマンドを実行。

```sh
git clone https://git01.codeplex.com/nuget
cd nuget
git checkout -b 2.6 origin/2.6 

patch -p1 < patch.diff

cd src/Core
xbuild

cp bin/Debug/NuGet.Core.dll  ~/Library/Application\ Support/XamarinStudio-4.0/LocalInstall/Addins/MonoDevelop.PackageManagement.0.5/NuGet.Core.dll
``` 

以上で、環境準備は終わり。

## 試す

1. Xamarin Studio で Portable Class Library を作成する
2. プロジェクト設定を見ると Xamarin.Android、Xamarin.iOS などがあるが、これらを**チェックしてOKしても適用されてない** ![img1](https://dl.dropboxusercontent.com/u/264530/qiita/using_pcl_on_xamarin_studio01.png)
3. Nuget Manager から mvvmcorss で検索して "MvvmCross - Hot Tuna  Starter Pack" を Add してもエラーになる。![img1](https://dl.dropboxusercontent.com/u/264530/qiita/using_pcl_on_xamarin_studio02.png)

うーん、ダメか？

## プロジェクトファイルをちょっと修正する

ポータブルクラスライブラリのプロジェクトファイル(xxx.csproj) をテキストエディタで開き、``<TargetFrameworkProfile>`` の値を ``Profile104`` に書き換える。(修正前は Profile1 になってた。なぜ Profile104 かと言えば、Visual Studio で作った PCL プロジェクトのプロファイルが 104 だったから、という程度の理解レベル)

```xml xxx.csproj
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProductVersion>10.0.0</ProductVersion>
    <SchemaVersion>2.0</SchemaVersion>
    <ProjectGuid>{50D8E04F-FDE0-4A65-B388-5698BEFE8DC5}</ProjectGuid>
    <ProjectTypeGuids>{786C830F-07A1-408B-BD7F-6EE04809D6DB};{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}</ProjectTypeGuids>
    <OutputType>Library</OutputType>
    <RootNamespace>Portable2</RootNamespace>
    <AssemblyName>Portable2</AssemblyName>
    <TargetFrameworkProfile>Profile104</TargetFrameworkProfile>    <--Here!!!!!
    <TargetFrameworkVersion>v4.0</TargetFrameworkVersion>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
    <DebugSymbols>true</DebugSymbols>
```

## 再度試す

もう一度プロジェクトを開いて Nuget から "MvvmCross - Hot Tuna  Starter Pack" を Add すると、成功する。プロジェクトツリーを見ると必要なDLLやソースコードが配置されている。


これで Xamarin Studio でも PCL が使えそう。
MvvmCross の Tutorial - [MvvmCross N+1 Table of Context](http://mvvmcross.wordpress.com/) を試せます。
