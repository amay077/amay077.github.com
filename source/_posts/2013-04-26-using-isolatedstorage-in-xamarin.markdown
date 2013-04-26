---
layout: post
title: "Xamarin.Android/iOS で IsolatedStorage を使う"
date: 2013-04-26 18:04
comments: true
categories: [Xamarin, Android, iOS, C#]
---
アプリの設定情報なんかを保存する時、Android では SharedPreference 、iOS では NSUserDefaults を使うわけですが、プラットフォーム毎にコード書くのめんどい！と思って
<!--more-->
<blockquote class="twitter-tweet" lang="ja"><p><a href="https://twitter.com/search/%23Xamarin">#Xamarin</a> さん、SharedPref/android と userDefaults/ios が共通のInterfaceで使えるコンポーネントが欲しいです</p>&mdash; あめい@ざまらーさん (@amay077) <a href="https://twitter.com/amay077/status/326942931028148224">2013年4月24日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

とツイートしたところ、Xamarin の中の人である [@atsushieno](https://twitter.com/atsushieno) さんから、

<blockquote class="twitter-tweet" data-conversation="none" lang="ja"><p>IsolatedStorageじゃダメなんでしょうか? RT @<a href="https://twitter.com/amay077">amay077</a>: <a href="https://twitter.com/search/%23Xamarin">#Xamarin</a> さん、SharedPref/android と userDefaults/ios が共通のInterfaceで使えるコンポーネントが欲しいです</p>&mdash; Atsushi Enoさん (@atsushieno) <a href="https://twitter.com/atsushieno/status/327355284416757761">2013年4月25日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

とアドバイスを頂きました。

IsolatedStorage(分離ストレージ) とは、.NET Framework(当然 Mono も)に用意されている、OS のファイルシステムとは切り離されたデータ領域の事で、アプリケーション毎、ユーザー毎など、アクセス権限を細かく設定できるのが特徴です。

そこで気になったのは、Xamarin.Android/iOS で IsolatedStorage を利用した時に、実体はどこに保存されるのか？ということ。

Android では、ストレージの ``/data/data/<アプリ名>`` 配下は、そのアプリ専用のデータ領域であり、そのアプリしかアクセス許可が与えられていない他、アプリをアンインストールするとそのデータ領域も削除されます。
(iOS は詳しくないですが、 userDefaults も同様だろうと思ってます。)


で、調べてみました。

## Xamarin.Android の場合

Xamarin Studio で、適当な Xamarin.Android プロジェクトを作って、MainActivity の onCreate で、IsolatedStorage にファイルを作成しています。
IsolatedStorage は、どこまでアクセス許可を与えるかを [``IsolatedStorageScope``](http://msdn.microsoft.com/ja-jp/library/system.io.isolatedstorage.isolatedstoragescope.aspx) 列挙体 で指定しますが、ここでは、SharedPreference の MODE_PRIVATE に最も近いであろう ``Application`` と ``User`` を組み合わせた Scope で生成する ``GetUserStoreForApplication()`` を使います。

サンプルコードは、

* [分離ストレージ（Isolated Storage）を使ってデータの保存と取得を行う – CH3COOH(酢酸)の実験室](http://ch3cooh.jp/index.php/tips/windowsphone7/system/strage/using-isolated-storage/)

を参考にさせてもらいました。

```c# MainActivity.cs
using System;

using Android.App;
using Android.Content;
using Android.Runtime;
using Android.Views;
using Android.Widget;
using Android.OS;
using System.IO.IsolatedStorage;
using System.IO;

namespace IsolatedStorageTest
{
    [Activity (Label = "IsolatedStorageTest", MainLauncher = true)]
    public class Activity1 : Activity
    {
        int count = 1;

        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);

            // Set our view from the "main" layout resource
            SetContentView(Resource.Layout.Main);

            var file = IsolatedStorageFile.GetUserStoreForApplication();
            
            // 分離ストレージにtest.txtというファイルを作成しストリームを開く
            using (IsolatedStorageFileStream strm = file.CreateFile("test.txt"))
            using (StreamWriter writer = new StreamWriter(strm))
            {
                // データを書き込む
                writer.Write("Hello!");
                writer.Write("Storage.");
            }
        }
    }
}
```

上記の ``var file = …`` の次の行にブレークポイントを設置して、デバッグ実行します。
ブレークしたら、変数 ``file`` をウォッチなどを覗いてみると、Non-public なフィールド ``directory`` が「/data/data/<アプリ名>/files/.config/.isolated-storage」
を示していることが分かります。

![image1](https://dl.dropboxusercontent.com/u/264530/qiita/location_of_isolatedstorage_in_xamarin1.png)

## Xamarin.iOS の場合

次に Xamarin.iOS でも適当なプロジェクトを作って、``ViewDidLoad`` に IsolatedStorage への書き出しコードを挿入します。

```c# IsolatedStorageiOSTestViewController.cs
using System;
using System.Drawing;

using MonoTouch.Foundation;
using MonoTouch.UIKit;
using System.IO.IsolatedStorage;
using System.IO;

namespace IsolatedStorageiOSTest
{
    public partial class IsolatedStorageiOSTestViewController : UIViewController
    {
        public IsolatedStorageiOSTestViewController() : base ("IsolatedStorageiOSTestViewController", null)
        {
        }
		
        public override void DidReceiveMemoryWarning()
        {
            // Releases the view if it doesn't have a superview.
            base.DidReceiveMemoryWarning();
			
            // Release any cached data, images, etc that aren't in use.
        }
		
        public override void ViewDidLoad()
        {
            base.ViewDidLoad();
			
            var file = IsolatedStorageFile.GetUserStoreForApplication();
            
            // 分離ストレージにtest.txtというファイルを作成しストリームを開く
            using (IsolatedStorageFileStream strm = file.CreateFile("test.txt"))
                using (StreamWriter writer = new StreamWriter(strm))
            {
                // データを書き込む
                writer.Write("Hello!");
                writer.Write("Storage.");
            }
        }
		
        public override bool ShouldAutorotateToInterfaceOrientation(UIInterfaceOrientation toInterfaceOrientation)
        {
            // Return true for supported orientations
            return (toInterfaceOrientation != UIInterfaceOrientation.PortraitUpsideDown);
        }
    }
}
```

デバッグして、``file`` 変数を覗いてみると、``directory`` が「<省略>/iPhone Simulator/6.1/Applications/<アプリのUUID>/Documents/.config/.isolated-storage」
を示していることが分かります。

![image2](https://dl.dropboxusercontent.com/u/264530/qiita/location_of_isolatedstorage_in_xamarin2.png)

ということで、IsolatedStorage の保存先は、Android では ``/data/data/<アプリ名>/``、iOS の場合は ``/<アプリのUUID>/Documents/`` と、アプリごとの固有の場所になっている事が分かりました。

## セキュリティ上の注意

Android の ``/data/data/<アプリ名>`` はアプリしかアクセスできないディレクトリですが、データ自体が暗号化されるわけではありません。(端末のROOT化や、apkを入手してエミュレータでアプリを実行することで /data/data/ のデータは取り出せます。)

また、iOS の ``/<アプリUIID>/`` はセキュアではないようです。(UUIDさえ分かれば他のアプリからもアクセスできるという事？)

* [How Not to Store Passwords in iOS](http://software-security.sans.org/blog/2011/01/05/using-keychain-to-store-passwords-ios-iphone-ipad)
* [iphone - How to secure NSUserDefaults? - Stack Overflow](http://stackoverflow.com/questions/1560801/how-to-secure-nsuserdefaults)
* [NSUserDefaults Secure? - iPhone Dev SDK](http://iphonedevsdk.com/forum/iphone-sdk-development/28041-nsuserdefaults-secure.html)

秘匿情報の保存には ``KeyChain`` を使え、と書いてありますね。

さらに、.NET の IsolatedStorage の説明にも、「暗号化されていないキーやパスワードは保存するな」と書いてあります。

* [Scenarios for Isolated Storage - MSDN](http://msdn.microsoft.com/en-us/library/3ak841sy.aspx)

> You **should not use** isolated storage in the following situations:
>
> * To store high-value secrets, such as unencrypted keys or passwords, because isolated storage is not protected from highly trusted code, from unmanaged code, or from trusted users of the computer.

(つか、[日本語サイト](http://msdn.microsoft.com/ja-jp/library/vstudio/3ak841sy.aspx#scenarios_for_isolated_storage)、誤訳ってない？)

ということで、パスワードなどの秘匿情報をどうしても端末に保存する時は、

* iOS なら KeyChain を使う
* Android の場合は独自の暗号化を施す(Mono の ``SecureString`` とか ``ProtectedData`` が使える？)

などの対策が必要です。

## まとめ

* IsolatedStorage は、SharedPreference や NSUserDefaults の代わりに、「アプリケーション情報格納領域」として使える
	* 本文に書きませんでしたが、アプリ専用領域なので、アプリをアンインストールするとちゃんと消えます
* ただし、パスワードとかの秘匿情報は保存しちゃダメよ
* [``IsolatedStorageSettings``](http://msdn.microsoft.com/ja-jp/library/system.io.isolatedstorage.isolatedstoragesettings) が使えたらもっと便利だったが、 System.Windows.dll が必要なのでムリー

## ちょっと気になる

* [分離ストレージ - MSDN](http://msdn.microsoft.com/ja-jp/library/3ak841sy)

に、

> 分離ストレージは Windows ストア の apps では使用できません。 代わりに、ローカル データとファイルを格納する Windows ランタイム API に含まれる Windows.Storage の名前空間にアプリケーション データのクラスを使用します。 詳細については、Windows Dev センターの" アプリケーション データ "を参照してください。

と書いてある。Windows 8 の Store App だと、IsolatedStorage が使えなくて、代わりに WinRT を使う必要があるらしい。それを考えると、IsolatedStorage を直で使わずに１枚咬ませた方が良さそう。