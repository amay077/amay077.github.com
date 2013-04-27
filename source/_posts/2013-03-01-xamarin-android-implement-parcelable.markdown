---
layout: post
title: "Xamarin.Android で画面遷移時にデータを渡す"
date: 2013-03-01 00:19
comments: true
categories: [Xamarin, Android, C#]
---
[前回のポスト](http://qiita.com/items/3232064cc8880c809aee)で HelloWorld を見てみましたが、次は Activity をもう一つ作って画面遷移してみます。

<!-- more -->

## Activity を追加する

メニュー → 新規 → ファイル と選択するとこんなダイアログが出るので Activity クラス名を入力して決定します。

!["new_file_dialog"](https://dl.dropbox.com/u/264530/qiita/xamarin_percelable_create_activity.png)

すると ``NextActivity.cs`` ができます。

```c# NextActivity.cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

using Android.App;
using Android.Content;
using Android.OS;
using Android.Runtime;
using Android.Views;
using Android.Widget;

namespace HelloXamarinAndroid
{
    [Activity (Label = "NextActivity")]			
    public class NextActivity : Activity
    {
        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);

            // Create your application here
        }
    }
}
```

作ってくれるのはここまでです。レイアウト用の Next.axml やリソースファイルは別途作成する必要があります。今回は必要ないのでスルーで。

Android本家だと今はウィザードで Javaソース、レイアウトXML、リソースファイルまで作ってくれてしかも AndroidManifest.xml にも書いてくれちゃいます、便利な世の中になりましたね。

ちなみに Xamarin.Android では AndroidManifest.xml への登録は必要ありません。

## 画面遷移してみる

画面遷移は本家と同じく Intent を使います。

```c# MainActivity.cs
button.Click += (sender, e) => 
{
    // Goto NextActivity
    var intent = new Intent(this, typeof(NextActivity));
    this.StartActivity(intent);
};
```

本家だと ``NextActivity.class`` としていたところは ``typeof(NextActivity)`` になってますね。

## 自作クラスを Intent に含める

さて、画面遷移時にデータを渡すという、よくある処理をやってみたいと思います。
データは ``Card`` というクラスにします。

```c# Card.cs
namespace HelloXamarinAndroid
{
    class Card
    {
        public string Name { get; set; }
        public string Phone { get; set; }
    }
}
```

C# なので、プロパティ作るのも楽でいいですね。getter/setter なんてアホらしくて…。
この Card クラスのインスタンスを、MainActivity から NextActivity に渡したいと思います。

## IParcelable を実装したいが？

Android では、Activity とか Service をまたぐオブジェクトは Percelable インターフェースを実装しなければならないのでした。
「じゃあ Xamarin では？」「IParcelable でしょ」というわけで ``IParcelable`` インターフェースを実装するわけですが、こいつが ``IDisposable`` や ``IJavaObject`` を継承していてこれらも実装させられます。

なんか違う・・・とあきらめてググったところ、こんなんでました。

* [Implementing IParcelable in Mono for Android » Dan Clarke](http://dan.clarke.name/2012/09/implementing-iparcelable-in-mono-for-android/)

以降、これに習うことにします。

### Mono.Android.Export を参照に追加する

いきなりこれを忘れて、謎のビルドエラーと１０分程格闘しました(汗)

!["assembly_reference"](https://dl.dropbox.com/u/264530/qiita/xamarin_assenbly_reference.png)

ソリューションツリーの参照の右クリックメニューからアセンブリ参照ダイアログが出るので、``Mono.Android.Export`` を見つけて追加します。

#### Free 版で使ってる人はここで「Trial するか、買うか、どっちかにせいや」と言われます。 ####
Free 版はプログラムのサイズに制限があるのですが、アセンブリを追加することでその制限を超えてしまうものと思われます。

### IParcelable を実装する

Card クラスを、``IParcelable`` を実装するのと共に ``Java.Lang.Object`` から派生させます。
あーそういうことですか。

```c# Card.cs
namespace HelloXamarinAndroid
{
    class Card  : Java.Lang.Object, IParcelable
    {
        public string Name { get; set; }
        public string Phone { get; set; }

        public Card(string name, string phone)
        {
            this.Name = name;
            this.Phone = phone;
        }

        #region IParcelable implementation

        public int DescribeContents()
        {
            return 0;
        }

        public void WriteToParcel(Parcel dest, ParcelableWriteFlags flags)
        {
            dest.WriteString(this.Name);
            dest.WriteString(this.Phone);
        }

        #endregion
    }
}
```

まだ何か足りません。あ、CREATOR だ。

## CREATOR フィールドを準備する

本家では ``public static final Parcelable.Creator<MyParcelable> CREATOR = …`` な感じで用意していましたが、Xamarin.Android では次の２点が異なります。

1. 匿名クラスが使えないので、``IParcelableCreator`` インターフェースを実装したクラスを用意する。
2. CREATOR フィールドの代わりに ``IParcelableCreator`` を返す static メソッドを用意し、``[ExportField("CREATOR")]`` 属性を付ける。

これらを実装すると、 Card クラスはこうなります。

```c# Card.cs
using Java.Interop;

namespace HelloXamarinAndroid
{
    class Card  : Java.Lang.Object, IParcelable
    {
        public string Name { get; set; }
        public string Phone { get; set; }

        public Card(string name, string phone)
        {
            this.Name = name;
            this.Phone = phone;
        }

        #region IParcelable implementation

        public int DescribeContents()
        {
            return 0;
        }

        public void WriteToParcel(Parcel dest, ParcelableWriteFlags flags)
        {
            dest.WriteString(this.Name);
            dest.WriteString(this.Phone);
        }

        #endregion

        [ExportField("CREATOR")]
        public static IParcelableCreator GetCreator()
        {
            return new MyParcelableCreator();
        }

        // Parcelable.Creator の代わり
        class MyParcelableCreator : Java.Lang.Object, IParcelableCreator
        {
            #region IParcelableCreator implementation
            Java.Lang.Object IParcelableCreator.CreateFromParcel(Parcel source)
            {
                var name = source.ReadString();
                var phone = source.ReadString();

                return new Card(name, phone);
            }

            Java.Lang.Object[] IParcelableCreator.NewArray(int size)
            {
                return new Java.Lang.Object[size];
            }
            #endregion
        }
    }
}
```

これでやっと、「受け渡し可能な」クラスが作成できました。

## 受け渡ししてみる

まず渡す方。Intent に詰めます。

```c# MainActivity.cs
button.Click += (sender, e) => 
{
    var card = new Card("amay", "987-654-3321");

    // Goto NextActivity
    var intent = new Intent(this, typeof(NextActivity));
    intent.PutExtra("card", card); // intent に Card を詰める

    this.StartActivity(intent);
};
```

次に取り出す方。Intent から取り出します。

```c# NextActivity.cs
namespace HelloXamarinAndroid
{
    [Activity (Label = "NextActivity")]
    public class NextActivity : Activity
    {
        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);

            // Create your application here

            var card = this.Intent.GetParcelableExtra("card") as Card;

            // Show toast
            Toast.MakeText(this, 
                String.Format("name:{0}, phone:{1}", card.Name, card.Phone), 
                ToastLength.Long).Show();
        }
    }
}
```

``〜 as Type`` 句は型が違っていたら null にしてくれる書き方です。
あと Toast。``.Show()`` は忘れずに。

これで実装完了。動かしてみます。

!["receive_parcel"](https://dl.dropbox.com/u/264530/qiita/xamarin_parcel_received.png)

以上、受け渡し可能なクラスの実装方法でした。

## まとめ
* Mono.Android.Export の追加を忘れない
* CREATOR フィールドの代わりに [ExportField("CREATOR")] 属性を付ける
* Java の匿名クラスは意外と便利だった
* せっかくプロジェクト作ったので GitHub に置いておきます - [amay077 / XamarinAndroid_ParcelableSample](https://github.com/amay077/XamarinAndroid_ParcelableSample/tree/Posted_Qiita)

## 参考
* [Implementing IParcelable in Mono for Android » Dan Clarke](http://dan.clarke.name/2012/09/implementing-iparcelable-in-mono-for-android/)
* [monodroid-samples/ExportAttribute at master · xamarin/monodroid-samples](https://github.com/xamarin/monodroid-samples/tree/master/ExportAttribute) なんだ、本家のサンプルあるじゃん
* [Limitations | xamarin](http://docs.xamarin.com/guides/android/advanced_topics/limitations) 制限だったり、制限が解除された情報が載ってるので要チェック

