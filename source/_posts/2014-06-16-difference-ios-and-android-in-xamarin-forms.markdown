---
layout: post
title: "Xamarin.Forms でどうにかしたい iOS と Android の違い"
date: 2014-06-10 15:30
comments: true
categories: [Xamarin, Android, iOS]
---
Xamarin.Forms で簡単な iOS/Android 両対応アプリを作ってみてて、悩ましい点がいくつか見つかってるので、挙げてみる。
<!--more-->

## ~~不可視の扱い~~

~~Forms 側のパーツには ``IsVisible = true | false`` がある。
iOS は ``true | false`` なのでいいけど、Android の Visibility は、 ``Visible | Invisible | Gone`` の3つある。~~

~~Forms 側での ``IsVisible = false`` は、Android では ``Invisible`` に相当するみたい。つまり StackLayout とかで「不可視なパーツが **詰められない**」。 iOS の ``Visible = false`` は **詰められる** 模様。~~

``IsVisible = false`` は Android ではちゃんと ``Gone`` になってました、すいませんでした。

## 空文字の扱い

IsVisible と勘違いしてたのはこっちだった。

StackLayout に、Label を2つ積んで、上の Label を空文字にすると、iOSでは詰められるけど、Androidでは空白が空くみたい。こっちはちゃんと裏をとった(汗)

https://gist.github.com/amay077/cf0f4ca1aa14d54bac9a


## 画面回転時の再構築

Android だと、画面を回転させると ``onCreate`` からやり直しなのは常識。
Forms アプリを Android で動かして回転させると、なんと **RootPage まで戻って** しまう。なんじゃこりゃ。

## 回避方法

<blockquote class="twitter-tweet" lang="ja"><p><a href="https://twitter.com/amay077">@amay077</a> あとAndroidの回転問題（バグです）はとりあえず ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation で回避するといいそうです（そりゃそうだ…）</p>&mdash; Atsushi Eno (@atsushieno) <a href="https://twitter.com/atsushieno/statuses/476645011602165760">2014, 6月 11</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

こんな感じっすね。

```csharp MainActivity.cs
[Activity(MainLauncher = true, 
    ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation )]
public class MainActivity : AndroidActivity
{
```

## BACKキーの扱い

iOS なら ``NavigationPage.SetHasNavigationBar(page, false)`` としてしまえば、ナビゲーションバーが表示されなくなるので、前の画面に戻ることはできなくなるが、Android の BACKキーを無効にするにはどうしたら。。。

### 自己解決

``AndroidActivity`` のサブクラスで、``OnBackPressed`` を override して実装を潰してしまえばよい。けど画面毎に「戻る／戻れない／Confirm出す」とか細かい制御ができるのかは不明。

## デフォルトスタイル

iOS は白基調、Androidは黒基調なので、Forms側で ``TextColor = Color.Black`` などとすると、当たり前だが Android で見えない。
iOS はスタイル変えるのしんどいので、Android側の Theme を ``Theme.Holo.Light`` にしとく。

```csharp MainActivity.cs
[Activity(Label = "MyApp",  
 MainLauncher = true, 
 Theme = "@android:style/Theme.Holo.Light")]
public class MainActivity : AndroidActivity
{
    protected override void OnCreate(Bundle bundle)
    {
        /* 以下省略 */
```

## 起動時

Android 側の起動時に ActionBar の付いた空白画面が表示される時間が割とながくて気になる(Galaxy Nexus だけど)。Forms の画面をロードするのに時間がかかるのだろうか？
ActionBar だけでも消したくて Theme を ``Theme.Holo.Light.NoActionBar`` にしてみたら Page が表示されなくなった。。。

NoActionBar な Theme を使うと Activity.ActionBar が null になるんだけど、Xamarin.Forms がそれに対応してない気がした（スタックトレース見ると UpdateActionBar で NullReferenceException だし）ので、[Bugzilla](https://bugzilla.xamarin.com/buglist.cgi?product=Forms&component=Forms&resolution=---&list_id=92025) に登録してみた、初めて。どうなるやら。

## 文字の自動縮小

iOS の ``UITextField`` は ``adjustsFontSizeToFitWidth `` を設定するとパーツのサイズに合わせて文字サイズを自動拡縮してくれる機能があったけど、Forms の ``Label`` にはそんなものはありません。``PageRenderer`` を使って iOS 独自処理しないとダメ。

### 自己解決

やはり PageRenderer 使うとできた → [Xamarin.Forms の Label から iOS の UILabel を取り出す](http://qiita.com/amay077/items/8eaa595cc2fc88797b2f)

## iPhone と iPad

StackLayout や RelativeLayout でUIを書けば、相対的な位置関係は iPhone と同じものが iPad でも再現されるが、サイズをリテラルで指定するところは、特にインテグレーションしてくれるわけでないので、プラットフォーム毎に調整が必要。例えば文字サイズは、iPad では iPhone より大きな値にしないと残念な感じに。


他にも見つけたら書きます。
