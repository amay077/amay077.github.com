---
layout: post
title: "Xamarin.Android で音楽の音量を下げてから効果音を再生する"
date: 2016-07-09  23:59:59 +0900
comments: true
categories: [C#, Android, Xamarin]
---
カーナビでよくある、音楽流しながらナビしてると、ガイダンス中は音楽のボリュームを一時的に下げて、案内の音声を再生するってやつ。

<!--more-->

* [スマホアプリ起動時(iOS/Android)に再生中の音楽を停止させる方法 - スタック・オーバーフロー](http://ja.stackoverflow.com/questions/27452/%e3%82%b9%e3%83%9e%e3%83%9b%e3%82%a2%e3%83%97%e3%83%aa%e8%b5%b7%e5%8b%95%e6%99%82ios-android%e3%81%ab%e5%86%8d%e7%94%9f%e4%b8%ad%e3%81%ae%e9%9f%b3%e6%a5%bd%e3%82%92%e5%81%9c%e6%ad%a2%e3%81%95%e3%81%9b%e3%82%8b%e6%96%b9%e6%b3%95/27459#27459)

に回答したので、その関連でやってみた。

```csharp MainActivity.cs
[Activity(Label = "AudioFocusSample", MainLauncher = true, Icon = "@mipmap/icon")]
public class MainActivity : Activity, AudioManager.IOnAudioFocusChangeListener
{
    protected override void OnCreate(Bundle savedInstanceState)
    {
        base.OnCreate(savedInstanceState);
        SetContentView(Resource.Layout.Main);

        var audioManager = (AudioManager)GetSystemService(Context.AudioService);

        // 先に効果音を読み込んでおく
        var soundPool = new SoundPool(1, Stream.Music, 0);
        var soundId = soundPool.Load(ApplicationContext, Resource.Raw.cat, 0);
        // SoundPool は再生完了のコールバックがないので、事前に長さを得ておく
        var duration = GetSoundDuration(Resource.Raw.cat);

        FindViewById<Button>(Resource.Id.buttonRequestFocus).Click += async (sender, e) => 
        {
            // ダッキングを許可する AudioFocus を要求
            var result = audioManager.RequestAudioFocus(this, Stream.Music, AudioFocus.GainTransientMayDuck);
            if (result == AudioFocusRequest.Granted)
            {
                // 効果音を再生する
                soundPool.Play(soundId, 1.0f, 1.0f, 0, 0, 1.0f);
                // 再生完了まで待つ
                await Task.Delay((int)duration);
                // AudioFocus を開放
                audioManager.AbandonAudioFocus(this);
            }
        };
    }

    // 音声の再生長さを得る
    private long GetSoundDuration(int rawId)
    {
        using (var player = MediaPlayer.Create(ApplicationContext, rawId))
        {
            return player.Duration;
        }
    }

    // IOnAudioFocusChangeListener の実装（RequestAudioFocus のために必要）
    public void OnAudioFocusChange([GeneratedEnum] AudioFocus focusChange)
    {
       // 今回は使用しない
    }
}
```

効果音 ``Resource.Raw.cat`` は、 [On-Jin ～音人～](http://on-jin.com/sound/index.php) さんのを使わせてもらいました。

音量を小さくする＝絞る＝ダッキング、というのがなかなか連想しづらいので辿りつけない。
Android の音声再生は、他に ``MediaPlayer`` とか ``AudioTrack`` もあるし、リモコンなど外部からの操作のため？に BroadcastReceiver を使わないといけないし。

サンプルアプリはこちら。

* [amay077/AudioFocusSample](https://github.com/amay077/AudioFocusSample)

次は Xamarin.iOS でやってみる。

## 参考

* [Managing Audio Focus | Android Developers](https://developer.android.com/training/managing-audio/audio-focus.html)
* [Android Tips #48 BGM や効果音を再生する ｜ Developers.IO](http://dev.classmethod.jp/smartphone/android/android-tips-48-soundpool-mediaplayer/)
* [Managing Audio Playback part2｜Android開発記録雑記](http://ameblo.jp/negiiiimo/entry-11488832997.html)
* [音を制御する - AudioManager - Qiita](http://qiita.com/KeithYokoma/items/3896f5934478fa560a50)
* [Fire TV でのオーディオフォーカスの管理 - アマゾン アプリ 開発者ポータル](https://developer.amazon.com/public/ja/solutions/devices/fire-tv/docs/managing-audio-focus) 
