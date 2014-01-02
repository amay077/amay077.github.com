---
layout: post
title: "Xamarin で Windows Azure モバイルサービスを使う(その3:プッシュ通知編)"
date: 2013-12-19 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#, Azure, BaaS]
---

たぶん最終回です。[前回](http://qiita.com/amay077/items/ea510071bee85569dc18) は、OAuth認証を実装しました。
今回は、前回のサンプルの続きとして、Xamarin.iOS + Azure Mobile Service によるプッシュ通知を実装してみます。
<!--more-->
プッシュ通知を組み込むためのチュートリアルはこちら

* [Get started with push notifications (Xamarin iOS) | Mobile Dev Center](http://www.windowsazure.com/en-us/develop/mobile/tutorials/get-started-with-push-xamarin-ios/?fb=ja-jp)

では、早速いってみましょう。

## プッシュ通知用の証明書ファイルを作成する

> 1. Generate the Certificate Signing Request file
> 2. Register your app and enable push notifications
> 3. Create a provisioning profile for the app

についてです。

iOS でプッシュ通知を行うには Apple Push Notification Service(APNS) を使用しますので、その準備から始まります。
「もう知ってるよ」という人は 「4. Configure Mobile Services」から読んでも良いかと思います。

初めて行う人は、こちら↓の説明の方が日本語で分かりやすいかと思います。

* [iPhoneのプッシュ通知(APNS)の証明書の作り方 | 三度の飯とエレクトロン](http://blog.katty.in/4040)

この手順で最後に必要なのは「p12ファイル」です。パスワードはかかっていても問題ありません。

## Azure モバイルサービスにプッシュの設定をする

> 4 Configure Mobile Services

についてです。

まず、Azure マネージメントポータル (https://manage.windowsazure.com/) から、モバイルサービス → 項目 → を選んで、上部にある「プッシュ」を選択します。
次に「apple プッシュ通知の設定」の証明書に、先に手順で入手した 「.p12ファイル」をアップロードします。パスワードはその時決めたものを入力します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_3_02.png)

## アプリケーションにプッシュ通知を実装する

> 5 Add push notifications to your app

について。ここ、ハマりどころ多いです。

Xamarin Studio で、前回のサンプルを開いてコードを追加していきます。

``AppDelegate.cs`` に、端末を識別するためのIDを保持するプロパティを作ります。

```csharp AppDelegate.cs
public string DeviceToken { get; set; }
```

``ToDoItem.cs`` にも同じく DeviceToken を追加します。
が、説明にある ``[DataMember…`` は間違いです。正しくは ``[JsonProperty…`` です。

```csharp ToDoItem.cs
×[DataMember(Name = "deviceToken")]
[JsonProperty(PropertyName = "deviceToken")]
public string DeviceToken { get; set; }
```

再び ``AppDelegate.cs`` に戻って、アプリが起動完了した時に、APNS サーバにアプリを登録するコードを追加します。

```csharp AppDelegate.cs
public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
{
    UIRemoteNotificationType notificationTypes = UIRemoteNotificationType.Alert | 
        UIRemoteNotificationType.Badge | UIRemoteNotificationType.Sound;
    UIApplication.SharedApplication.RegisterForRemoteNotificationTypes(notificationTypes); 

    return true;
}
```

次に同じく  ``AppDelegate.cs`` に、アプリ登録完了時にデバイストークンを受け取るコールバックを実装します。

**ここが2つ目のハマりポイントです。**
``deviceToken.Description`` を使用していますが、これは iOSネイティブと異なり **空白入りの文字列を返す** ようです。そのためそのまま DeviceToken として利用すると、通知が来ません。エラーにもならないので原因究明に数時間要しました…。``Replace`` で空白も取り除きます。

```csharp AppDelegate.cs
public override void RegisteredForRemoteNotifications(UIApplication application, NSData deviceToken)
{
    string trimmedDeviceToken = deviceToken.Description;
    if (!string.IsNullOrWhiteSpace(trimmedDeviceToken))
    {
        trimmedDeviceToken = trimmedDeviceToken.Trim('<');
        trimmedDeviceToken = trimmedDeviceToken.Trim('>');
        trimmedDeviceToken = trimmedDeviceToken.Replcae(“ “, “”); // ←追加
    }
    DeviceToken = trimmedDeviceToken;
}
```

そして、``AppDelegate.cs`` に、プッシュ通知受信時のコードを書きます。

```csharp AppDelegate.cs
public override void ReceivedRemoteNotification(UIApplication application, NSDictionary userInfo)
{
    Debug.WriteLine(userInfo.ToString());
    NSObject inAppMessage;

    bool success = userInfo.TryGetValue(new NSString("inAppMessage"), out inAppMessage);

    if (success)
    {
        var alert = new UIAlertView("Got push notification", inAppMessage.ToString(), null, "OK", null);
        alert.Show();
    }
}
```

最後に ``QSTodoListViewController.cs`` の ``OnAdd`` を次のように変更して、追加するデータに DeviceToken を含めるようにします。この値を使って、Azure 側でプッシュ通知を送ります。

```csharp QSTodoListViewController.cs
var deviceToken = ((AppDelegate)UIApplication.SharedApplication.Delegate).DeviceToken;

var newItem = new ToDoItem() 
{
    Text = itemText.Text, 
    Complete = false,
    DeviceToken = deviceToken
};
```

コードの記述はこれで終わりですが、Xamarin Studio から、iOS アプリを実機で動かしたことがない人は、iOS Developer Account の設定をしておきましょう。

システムメニュー → Preferences、Environment → Developer Accounts から追加しておきます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_3_03.png)

## マネージメントポータルで、データの追加時のスクリプトを登録する

> 6 Update the registered insert script in the Management Portal

についてです。

まず、マネージメントポータル https://manage.windowsazure.com/ を開き、モバイルサービス → サービス名 → データ → テーブル名 と選択します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_3_04.png)

次に、スクリプト を選択し、ドロップダウンから「挿入」を選択します。テーブルにデータが追加された時に実行するスクリプトが表示されます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_3_05.png)

スクリプトを以下に置き換えます。

```js
function insert(item, user, request) {
    request.execute();
    // Set timeout to delay the notification, to provide time for the 
    // app to be closed on the device to demonstrate toast notifications
    setTimeout(function() {
        push.apns.send(item.deviceToken, {
            alert: "Toast: " + item.text,
            payload: {
                inAppMessage: "Hey, a new item arrived: '" + item.text + "'"
            }
        });
    }, 2500);
}
```

これにより、データの追加から 2.5秒後に、APNS にプッシュ通知のリクエストを投げます。

## 動かす

> 7 Test push notifications in your app

やっと試せます。

Xamarin Studio から、実機で実行します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_3_06.png)

アプリ起動直後、通知の受信を許可するかどうかを尋ねられますので「Yes」で。(一度答えると二度と出ないのでしょうか？なのでチュートリアルの画像で。)

![](http://www.windowsazure.com/media/devcenter/mobile/mobile-quickstart-push1-ios.png)

Twitter 認証後、適当に項目を追加します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_3_07.png)

しばらく待っていると…

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_3_08.png)

無事、プッシュ通知を受信できましたー。
(実はここに辿り着くまで数十回試しているので最初に通知を受け取った時の感激と言ったら…)

まあ、たぶんどっかでハマるだろうなーと思いながら、やっぱりハマった！という感じでしたので、皆さんも最初はどこかでハマるんじゃないかと。

ハマりどころは文中にも書きましたが、下にもトラブルシューティングとして書きました。何かのお役に立てば。

## トラブルシューティング

### iOS Dev Center で Bundle ID 入力したらエラーになった

プロジェクト名に アンダースコア が含まれてるとダメです。
Xamarin Studio のプロジェクト プロパティ → iOS Application で直しましょう。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_3_09.png)

### アプリが起動直後(認証を通過した後)に落ちるようになった

[チュートリアル](http://www.windowsazure.com/en-us/develop/mobile/tutorials/get-started-with-push-xamarin-ios/?fb=ja-jp)にある、

```csharp
[DataMember(Name = "deviceToken")]
public string DeviceToken { get; set; }
```

は間違いです。下記が正しいです。

```csharp
[JsonProperty(PropertyName = "deviceToken")]
public string DeviceToken { get; set; }
```

### クライアントで「追加」しても、プッシュ通知が来ません

まずは、Azure マネージメントポータルで「ログ」を見てみましょう。
下記からアクセスできます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_3_01.png)

データ挿入時のスクリプトでエラーが出ていれば、ここに出力されるはずです。スクリプト内で ``console.log`` した内容もここに出力されます。

### それでもまだ、プッシュ通知が来ません

[チュートリアル](http://www.windowsazure.com/en-us/develop/mobile/tutorials/get-started-with-push-xamarin-ios/?fb=ja-jp)にある、

```csharp
public override void RegisteredForRemoteNotifications(UIApplication application, NSData deviceToken)
{
    string trimmedDeviceToken = deviceToken.Description;
    if (!string.IsNullOrWhiteSpace(trimmedDeviceToken))
    {
        trimmedDeviceToken = trimmedDeviceToken.Trim('<');
        trimmedDeviceToken = trimmedDeviceToken.Trim('>');
    }
    DeviceToken = trimmedDeviceToken;
}
```

は、コードが足りません。これでは ``trimmedDeviceToken`` の文字列内に空白が含まれてしまい、正しい DeviceToken にならないようです。

```csharp
public override void RegisteredForRemoteNotifications(UIApplication application, NSData deviceToken)
{
    string trimmedDeviceToken = deviceToken.Description;
    if (!string.IsNullOrWhiteSpace(trimmedDeviceToken))
    {
        trimmedDeviceToken = trimmedDeviceToken.Trim('<');
        trimmedDeviceToken = trimmedDeviceToken.Trim('>');
        trimmedDeviceToken = trimmedDeviceToken.Replcae(“ “, “”); // ←追加
    }
    DeviceToken = trimmedDeviceToken;
}
```

と、空白を除去する必要があります。

### しかしそれでもまだ、プッシュ通知が来ません

最初の手順、キーチェーンアクセスで CSR を作る時、「ユーザーのメールアドレス」「通称」を iOS Developer Center に登録しているものと同じにする必要があるみたいです。これが間違っていると、.p12 ファイルを書き出す時に選択すべき「Apple Development IOS Push Services: net.azure-mobile.amay077-baas-test」という鍵が見つからない(文字化けしてる)はずです。

また、iOS Developer Center に .p12 ファイルを再登録するとプロビジョニングファイルを再作成→端末に更新する必要があります。

これらを忘れると通知が来ません。そしてエラーにもならないので厄介です。

## 雑感

しかし [Azure のチュートリアル](http://www.windowsazure.com/en-us/develop/mobile/tutorials/get-started-with-push-xamarin-ios/?fb=ja-jp#add-push)、これ一度も試してないんじゃないの？

クラス名の接頭辞や大文字小文字の間違いはまだかわいいものだけど、ここミスると致命的という点が２つも。一度辿れば気づくと思うんだけどなあ。
最後にある completed example project ではちゃっかり正しいコードだったりするので、コンテンツの修正漏れなんだろうか。。。

それはともかく、何かトラブルがあると Xamarin のようなネイティブに皮を被せてるプラットフォームは途端に不安になります。どこに原因があるのか追求しづらいから。

今回も、コードの修正後、いきなりアプリが落ちるようになって、まずα版を使っていたのでそれを疑って stable に戻して、次に実機が悪いのかとシミュレータで動かしてみたり、いろいろ試行錯誤してやっとチュートリアルの誤記だと分かったり。まだノウハウが無いといえばそれまで、α版が自己責任なのはその通りですが、イレギュラー時のリスクはあるなあと思いました。
