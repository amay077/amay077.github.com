---
layout: post
title: "Xamarin.Forms のバージョンを 1.2 から 2.3 に上げた時のエラー対処法"
date: 2016-07-06 23:59:00 +0900
comments: true
categories: [Xamarin, Xamarin.Forms, C#]
---
プロジェクトで使ってる Xamarin.Forms のバージョンを「1.2.3.6257」から「2.3.0.107」に上げたら、ビルドでエラーが発生するようになった。
<!--more-->

ググッてみると、「Xamarin Studio を再起動」とか「ソリューションを開き直せ」とか書いてある。
ホンマかいな？と思いながら実施すると、確かになんか変化が。

エラーメッセージが変わっただけだけど。

> Error initializing task XamlG: Not registered task XamlG.

とか、

> Error initializing task FixedCreateCSharpManifestResourceName: Not registered task FixedCreateCSharpManifestResourceName.

調べたときに見た情報の中には .csproj ファイルがどーのこーの書いてあったので、開いてみると、何やら以前のバージョンが残ってた。

ので、その行を削除してソリューションを開き直し、ビルドしたらエラーはなくなった。

```xml xxx.csproj
   <Import Project="$(MSBuildExtensionsPath32)\Microsoft\Portable\$(TargetFrameworkVersion)\Microsoft.Portable.CSharp.targets" />
これ削除→  <Import Project="..\packages\Xamarin.Forms.1.2.3.6257\build\portable-win+net45+wp80+MonoAndroid10+MonoTouch10\Xamarin.Forms.targets" Condition="Exists('..\packages\Xamarin.Forms.1.2.3.6257\build\portable-win+net45+wp80+MonoAndroid10+MonoTouch10\Xamarin.Forms.targets')" />
   <Import Project="..\packages\Microsoft.Bcl.Build.1.0.21\build\Microsoft.Bcl.Build.targets" Condition="Exists('..\packages\Microsoft.Bcl.Build.1.0.21\build\Microsoft.Bcl.Build.targets')" />
   <Import Project="..\packages\Xamarin.Forms.2.3.0.107\build\portable-win+net45+wp80+win81+wpa81+MonoAndroid10+MonoTouch10+Xamarin.iOS10\Xamarin.Forms.targets" Condition="Exists('..\packages\Xamarin.Forms.2.3.0.107\build\portable-win+net45+wp80+win81+wpa81+MonoAndroid10+MonoTouch10+Xamarin.iOS10\Xamarin.Forms.targets')" />
```

エラーは、ね。

## 参考

* [Xamarin Studio がビルドエラーでビルドできなくなった話 | rksoftware](https://rksoftware.wordpress.com/2016/04/24/001-16/)
* [xamarin - error : Error initializing task XamlG: Not registered task XamlG - Stack Overflow](http://stackoverflow.com/questions/27873185/error-error-initializing-task-xamlg-not-registered-task-xamlg)
* [Error compiling Xamarin Forms new project - Stack Overflow](http://stackoverflow.com/questions/34501301/error-compiling-xamarin-forms-new-project)
