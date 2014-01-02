---
layout: post
title: "Xamarin で Windows Azure モバイルサービスを使う(その１)"
date: 2013-12-14 0:00
comments: true
categories: [Xamarin, XAC13, iOS, Android, C#, Azure, BaaS]
---
Xamarin で BaaS を使うことについて、 [koji_yusa さん](http://qiita.com/koji_yusa/items/a6878bef10577ee744b5)や [kochizufan さん](http://qiita.com/kochizufan/items/c91b3a59a56d8fc54bb7) が、Parse の使用方法について書いてくれました。
<!--more-->
BaaS の中では Parse が一番知名度が高いでしょう。しかし！Microsoft も BaaS を提供しています。

Azure の名を冠しているため、「どうせ IaaS でしょ」とスルーする人が多いんじゃないかと思うので、今回は Microsoft の BaaS である「Azure モバイルサービス」について書きます。

## Azureモバイルサービス vs Parse

* [Windows Azure モバイルサービス - 料金詳細](http://www.windowsazure.com/ja-jp/pricing/details/mobile-services/)
* [Parse - Pricing](https://parse.com/plans)

実際のところは、Parse に比べて Azure の方がスペックは低いです。無料版では、API呼び出し回数は、Azure が50万回に対して、Parse は100万回、ストレージは、Azyreが20MB(!)に対して Parse が1GB などです。

唯一、Azure にのみある機能は「スケジュールされたジョブ」、いわゆる cron みたいなジョブの定期呼び出し機能です。無料版では1時間に1回の利用に限定されるようですが、日時処理には十分使えるでしょう。

[Nifty の BaaS](http://mb.cloud.nifty.com/price.htm) と比べても劣勢ですね、あれれ？

## Azure モバイルサービスを Xamarin から使う

Microsoft と Xamarin との提携により、Xamarin から Azure モバイルサービスは、簡単に利用することができます。提携前からライブラリの提供など対応は充実していましたが、提携により Microsoft のサイトでチュートリアルが公開されるなど、より充実しました。

そのチュートリアルを辿ってみます。

ちなみに環境は Mac + Xamarin Studio です。Win + Visual Studio でも同じ手順ですが、iOS 用のサンプルなので、iOS ならビルドと実行の為に Mac が必要です。Win しかないなら Android 用に置き換えて試せます。

### 1. Windows Azure にサインアップする

モバイルサービスを利用するには、まず Windows Azure に登録しなければなりません。本人確認のために、クレジットカードや携帯電話番号が必要になるのが煩わしいかもですが、勝手に請求されたりはしませんのでご安心を。

手順は↓が詳しいので割愛します。

* [Windows Azureに登録する | 初心者でも30分でできる　ビジネスで使える！WordPressでWebサイト](http://wordpress-web.azurewebsites.net/guide)

### 2. モバイルサービスを作成する

サインアップできたら Windows Azure マネージメントポータルを開きます。迷ったらここ。アドレスは、

* https://manage.windowsazure.com/

です。

下のような画面になるので、左メニューから モバイルサービス → 新しいモバイル サービスを作成する と進みます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_1_01.png)

URL に任意のIDを入力します(世界で一意になる必要があります)。また、地域を「東アジア」にします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_1_02.png)

続いて SQL Server の設定をします。ログイン名に任意のユーザー名、パスワードに任意のパスワードを設定します。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_1_03.png)

ウィザードを終わると、マネージメントポータルに戻ります。しばらくの「作製中…」の後、状態が「準備完了」となり、これでモバイルサービスは作成完了です。

### 3. Xamarin.iOS アプリケーションからモバイルサービスを使ってみる

作成したモバイルサービスをクリックします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_1_04.png)

ここから、一気です。

1. まずプラットフォームで「Xamarin」を選択し、
2. 「新しい XAMARIN アプリケーションを作成する」を展開、
3. 「TodoItem テーブルを作成する」をクリックして「作成されました」となるまで待ち、
4. Xamarin.iOS 用のサンプルアプリケーションをダウンロードします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_1_05.png)

ダウンロードした zip ファイルを解答し、Xamarin Studio で開きます。
参照 や Components を見ると、Azure Mobile Service 用のライブラリが組み込まれている事が分かります。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_1_06.png)

Debug で iPhone シミュレータで動かしてみます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_1_07.gif)

上の動画のように適当なアイテムを追加した後、ブラウザのマネージメントポータルで追加したデータを確認してみます。

上部のメニュー から データ → TodoItem と進むと、追加されたデータが確認できます。

![](https://dl.dropboxusercontent.com/u/264530/qiita/using_azure_mobile_service_by_xamarin_1_08.gif)

以上です。Xamarin.iOS で Azure モバイルサービスにデータを登録するサンプルが手に入りました。

### 4. サンプルのコードを眺めてみる

これで終わってもアレなので、コードを見てみます。
まず前述した Azure Mobile Service 用のライブラリが組みこまれていますが、これは、 Xamarin の Components ストアで提供されている、

* [Azure Mobile Services / Components / Xamarin](http://components.xamarin.com/view/azure-mobile-services)

です。これを主に使っているのは ``QSTodoService.cs`` です。

``QSTodoService.cs`` を見てみると、まず NameSpace から。

```csharp
using Microsoft.WindowsAzure.MobileServices;
```

次にコンストラクタなど。

``MobileServiceClient`` がメインクラスですが、これを使う前に ``CurrentPlatform.Init()`` が必要です(DI だか IoC だかですかね)。

そして ``MobileServiceClient.GetTable<ToDoItem>()`` をすることで Azure 上のテーブルを取得しています。まだデータは読み込みません。

```csharp
MobileServiceClient client;
IMobileServiceTable<ToDoItem> todoTable;

/**省略*/

QSTodoService ()
{
	CurrentPlatform.Init ();

	// Initialize the Mobile Service client with your URL and key
	client = new MobileServiceClient (applicationURL, applicationKey, this);

	// Create an MSTable instance to allow us to work with the TodoItem table
	todoTable = client.GetTable <ToDoItem> ();
}
```

データの読み込みは、``RefreshDataAsync`` にて。さりげなく async で。


```csharp
public List<ToDoItem> Items { get; private set;}

async public Task<List<ToDoItem>> RefreshDataAsync()
{
	try {
		// This code refreshes the entries in the list view by querying the TodoItems table.
		// The query excludes completed TodoItems
		Items = await todoTable
			.Where (todoItem => todoItem.Complete == false).ToListAsync ();

	} catch (MobileServiceInvalidOperationException e) {
		Console.Error.WriteLine (@"ERROR {0}", e.Message);
		return null;
	}

	return Items;
}
```

今回はこんなところで。

次回は、 Azure モバイルサービスの続きで、このサンプルに認証周りの機能を実装してみます。