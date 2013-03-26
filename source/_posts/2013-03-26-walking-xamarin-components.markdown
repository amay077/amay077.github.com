---
layout: post
title: "Xamarin Component Store を眺めてみる"
date: 2013-03-26 22:26
comments: true
categories: [Xamarin, C#]
---
Xamarin には Component Store というものがあって、Xamarin で利用できる部品をここから購入することができます。(FREE もあります)
<!--more-->
こちらです

* [Components / Xamarin](http://components.xamarin.com/)

!["1"](https://dl.dropbox.com/u/264530/qiita/walking_xamarin_components_1.png)

TAGS のところで対応プラットフォームを絞りこめます。
iOS は MonoTouch の歴史が長いだけあってなかなかの数のコンポーネントがありますが、Android, Windows に関してはまだこれから、という感じでしょうか。
iOS/Android 両対応のコンポーネントも1ページに収まる程度の数はあります。


## 試しにつかってみる
試しに

* [Alert Center / Components / Xamarin](http://components.xamarin.com/view/alert-center/)

!["2"](https://dl.dropbox.com/u/264530/qiita/walking_xamarin_components_2.png)

を使ってみます。

確か、Component Store は Xamarin Studio と連携してるとのことなので、そちらから使ってみましょう。

### プロジェクトの作成

Android Application、名称は **AlertCenterSample** としました。

### コンポーネントを (Nu)Get!

メニュー → プロジェクト → Edit Components… から。

!["3"](https://dl.dropbox.com/u/264530/qiita/walking_xamarin_components_3.png)

こんな画面になりました。次に **Open Conponent Store** をクリック。

!["4"](https://dl.dropbox.com/u/264530/qiita/walking_xamarin_components_4.png)

Xamarin Components が出ました。Webサイトと同じやつです
。この中から **Alert Center** をクリック。

!["5"](https://dl.dropbox.com/u/264530/qiita/walking_xamarin_components_5.png)

なんか動画が真っ黒ですが。気にせず **Add to App** をクリック。

!["6"](https://dl.dropbox.com/u/264530/qiita/walking_xamarin_components_6.png)

ソリューションエクスプローラの Components に Alert Center が追加されました。また Getting Started が表示されています。

これで部品が追加できたようです、簡単でした。ちなみにコレ、NuGet というものが使われているそうです。(私は VS2005 以前の人なので使った事がありません)

!["7"](https://dl.dropbox.com/u/264530/qiita/walking_xamarin_components_7.png)

ここから、Getting Started に表示されている通りに実装して動かしてみます。

### AndroidManifest に権限を追加する
最初、完全に見落としていましたが、AndroidManifest.xml に ``SYSTEM_ALERT_WINDOW`` 権限を追加してください、と書いてありました。
Xamarin Studio での AndroidManifest.xml 編集については、

* [Xamarin.Android で PERMISSION を設定する - Experiments Never Fail](http://amay077.github.com/blog/2013/03/02/xamarin-android-permission/)

をご参考に。

確かに、``SYSTEM_ALERT_WINDOW`` もありますね。

!["9"](https://dl.dropbox.com/u/264530/qiita/walking_xamarin_components_9.png)

この手順を飛ばすと、ボタンをクリックした瞬間にアプリが落ちます(経験者

### コーディング

Getting Started のコードをコピペで。

```c# MainActivity.cs
<省略>
using Xamarin.Controls;

namespace AlertCenterSample
{
    [Activity (Label = "AlertCenterSample", MainLauncher = true)]
    public class Activity1 : Activity
    {
        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);

            // Set our view from the "main" layout resource
            SetContentView(Resource.Layout.Main);

            // Get our button from the layout resource,
            // and attach an event to it
            Button button = FindViewById<Button>(Resource.Id.myButton);
			
            button.Click += delegate
            {
                AlertCenter.Default.Init (Application);
                
                AlertCenter.Default.PostMessage ("Knock knock!", "Who's there?", Resource.Drawable.Icon);
                AlertCenter.Default.PostMessage ("Interrupting cow.", "Interrupting cow who?",
                                                 Resource.Drawable.Icon, () => {
                    Console.WriteLine ("Moo!");
                });
            };
        }
    }
}
```

そういえば、コンポーネントを取得直後、なぜか Xamarin Studio さんがそれを認識してくれずエラーが出ていました(ビルドはできた)が、Xamarin Studio を再起動したらエラーは消えました。

!["8"](https://dl.dropbox.com/u/264530/qiita/walking_xamarin_components_8.png)

### 動かす

ビルドして実行するだけです。

!["10"](https://dl.dropbox.com/u/264530/qiita/walking_xamarin_components_10.png)

### iOS/Android 両対応と言っても 'Write once' ではない(かも知れない)
Alert Center は、iOS/Android 両対応とされていますが、Getting Started の iOS と Android のコードをよく見るとわかりますが、異なっています。「Android は iOS に '似せて' いる」とも書かれています。
同じコードで iOS/Android 共に動く！という夢はやっぱり見ない方がいいです。クロスプラットフォームで開発する時は View の部分はプラットフォーム毎にプロジェクトを分けなければならないので、共有できるコードも少ないと思います。

他のコンポーネントも、UI に絡むものは Look&Feel が統一できるだけで、(コンポーネントを使った)実装はプラットフォーム毎に必要と考えた方が良いでしょう。

## まとめ
Xamarin Studio に統合された Component Store はとても簡単に使うことができました。

試しに使った Alert Center の他にも、

* [Mobile Services by Windows Azure](http://components.xamarin.com/view/azure-mobile-services/)
* [Json.NET](http://components.xamarin.com/view/json.net)
* [Xamarin.Auth](http://components.xamarin.com/view/xamarin.auth)
* [Xamarin.Mobile](http://components.xamarin.com/view/xamarin.mobile)
* [Xamarin.Social](http://components.xamarin.com/view/xamarin.social)
* [ZXing.Net.Mobile](http://components.xamarin.com/view/zxing.net.mobile)

などは、どこかで使えそうです。

また、Component Store でなくても、.NET framework の基本的なクラスのみで書かれた(**POCO** な)ライブラリであれば、Xamarin でも概ね使うことができます。
例えば、Amazon Web Services の .NET SDK

* [AWS SDK for .NET | アマゾン ウェブ サービス（AWS 日本語）](http://aws.amazon.com/jp/sdkfornet/)

は、Xamarin.Mac で使うことができました。(Xamain.iOS/Android は未確認)

View 部分の共有は最初から想定しない思想なので、Model 部分を開発する際に使えるコンポーネントがさらに充実していくことに期待します。
