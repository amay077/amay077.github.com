---
layout: post
title: ".NET の MVVM + Messenger パターンにあこがれて、Java で Messenger クラスを自作してみた"
date: 2012-10-03 23:02
comments: true
categories: [Java, Android]
---
MVVM + Messenger パターンとは、

* [いまさら聞けない「MVVM + Messenger パターン」超入門 - present](http://tnakamura.hatenablog.com/entry/20110218/mvvm_messenger)

らへんを参照。

<!-- more -->

勉強がてら、Android(Java) で、Messenger を実装してみようとした。
要は、View 側で regist された Action<Message> を溜めておき、ViewModel 側から send(new Message()) した時に invoke させればいいんでしょ？と。

とりあえずこんな感じになると思われる。

```java common_interfaces
public interface Message {
}

public interface Action1<T> {
	public void invoke(T arg);
}
```

```java Messenger.java
public final class Messenger {
	
	private Map<String, Action1<? extends Message>> _actions = 
			new HashMap<String, Action1<? extends Message>>();

	public <T extends Message> void register(Action1<T> action) {
		String nameOfT = /* TODO action から T を取得 */
		_actions.put(nameOfT, action);
	}

	public void send(Message message) {
		/* TODO message の型をキーにして _actions から取り出して invoke */
	}
}
```

これの実装中、いくつか問題にハマった。

##問題1:X\<T> の T が取れない

Action<Message> を溜める時に、Message をキーにして Map に入れておけばいいでしょ、と思ったのだが、できない。
``Action<T>`` の ``T`` が取り出せない。

よく調べてみると

* [Java総称型メモ(Hishidama's Java Generics Memo)](http://www.ne.jp/asahi/hishidama/home/tech/java/generics.html#erasure)
* [イレイジャではジェネリクスの何が消えるのか](http://blogs.wankuma.com/nagise/archive/2008/10/13/158708.aspx)

だそうです。ふむーなるほど、実行時には T は消えてしまっていると。

しかしいろいろ試していたら、こんな方法で文字列としては取り出すことができました。

```java Messenger.java
public final class Messenger {
	
	private Map<String, Action1<? extends Message>> _actions = 
			new HashMap<String, Action1<? extends Message>>();

	public <T extends Message> void register(Action1<T> action) {
		// action が使ってる Generics な型を取り出す(という意味？)。
		// action.getClass().getInterfaces(); でもいけるかと思ったら、Action1 までしか取り出せなかった。
		Type[] types = action.getClass().getGenericInterfaces();
		// 文字列化したら Action1<T> の T の部分も実際の型名が得られた。
		// ex: "hoge.mvvm.Action1<com.piyo.MyMessage>"
		String typeString = types[0].toString();

		// < > 内だけ取り出す		
		int start = typeString.indexOf("<");
		int end = typeString.lastIndexOf(">");
		String nameOfT = typeString.subSequence(start + 1, end).toString();

		_actions.put(nameOfT, action);
	}

	public void send(Message message) {
		/* TODO message の型をキーにして _actions から取り出して invoke */
	}
}
```

##問題2:\<? extend T> or \<? super T> ?

次の問題、今度は send の方。
``_actions.get(nameOfMessage)`` で取り出した ``Action1<? extend Message>`` は、invoke メソッドの型が ``null`` になってて、使えませんでした。
なので仕方なく、Generics パラメータなしの ``Action1`` で受けることに。

```java Messenger.java
public final class Messenger {
	
	private Map<String, Action1<? extends Message>> _actions = 
			new HashMap<String, Action1<? extends Message>>();

	public <T extends Message> void register(Action1<T> action) {
		// action が使ってる Generics な型を取り出す(という意味？)。
		// action.getClass().getInterfaces(); でもいけるかと思ったら、Action1 までしか取り出せなかった。
		Type[] types = action.getClass().getGenericInterfaces();
		// 文字列化したら Action1<T> の T の部分も実際の型名が得られた。
		// ex: "hoge.mvvm.Action1<com.piyo.MyMessage>"
		String typeString = types[0].toString();

		// < > 内だけ取り出す		
		int start = typeString.indexOf("<");
		int end = typeString.lastIndexOf(">");
		String nameOfT = typeString.subSequence(start + 1, end).toString();

		_actions.put(nameOfT, action);
	}

	@SuppressWarnings("unchecked")
	public void send(Message message) {
		final String messengerTypeName = message.getClass().getName();
		
		if (!_actions.containsKey(messengerTypeName)) {
			return;
		}

		// Action1<? extends Message> だと、invoke の型が null になってしまう。
		// が、_actions は追加/取得を兼ねているので Action1<? super Message> にすることもできず…		
		// 仕方なく Generics 未使用で。
		@SuppressWarnings("rawtypes")
		Action1 action = _actions.get(messengerTypeName);
		action.invoke(message);
	}
}
```

とりあえず動くけど、なんかスッキリしない。。。

##ここまで実装しておいて…

* [MVVM：Messengerを理解するために自作してみた(2):Gushwell's C# Dev Notes](http://gushwell.ldblog.jp/archives/52146816.html)

を発見。
あれ、Map じゃなくて List でしたか。1回の Send で複数の Callback が走るのね。ま、いいや Android で使うだけだし。
.NET はいいなあ ``typeof(T)`` が使えて。

##使い方
だいたいこんな感じで使える。

```java usage
/** ダイアログを表示させるメッセージ */
public class DialogMessage implements Message {
	public String message;
	public Action1<Boolean> callback; 
	
	public DialogMessage(String message, Action1<Boolean> callback) {
		this.message = message;
		this.callback = callback;
	}
}

/** View側 */
public class MyActivity extends Activity {

	private MyViewModel _vm = new MyViewModel();

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my);
    	
		// Messenger に Message に対応する Action を登録する。
    	Messenger messenger = _vm.getMessenger();
    	messenger.register(new Action1<DialogMessage>() {
			@Override
			public void invoke(DialogMessage dlgMsg) {
				boolean isOk = showDialog(dlgMsg.message); // ホントは非同期なのでもう少し複雑
				if (isOk) {
					finish();
				}
				dlgMsg.callback.invoke(isOk); // VM に結果を通知する
			}
		});
    }

	// 終了ボタンが押されたら Finish コマンドを実行。
	public void exitButton_Click(View view) {
		_vm.commandExit.execute();
	}
}

/** ViewModel ※ Command インターフェースの定義とかは省略 */
public class MyViewModel {
	private final Messenger _messenger = new Messenger();

	public Messenger getMessenger() {
		return _messenger;
	}

	public final Command commandExit = new Command() {
		@Override
		public void execute() {
			// ダイアログを表示する Message を送る
			_messenger.send(new DialogMessage("終了します", new Action1<Boolean>() {
				@Override
				public void invoke(Boolean pushOk) {
					// ダイアログの表示結果を受ける
					Log.d(TAG, "Pushed button is OK? -> " + pushOk);
				}
			}));
		}
	};
}
```


##その他
* .NET の Messenger はなぜ Singleton なんだろう？VM と View の関係が 1:n だから？
* Messenger に否定的な見解もあるようで。確かに「View の変化は ViewModel の状態変化でのみ行われるべき」という主張にも一理ある。 → [トロこんぶ: MVVMでメッセンジャーを使わずにダイアログを表示する(MVVM Dialog Behaviorライブラリ提供)](http://torokonbu.blogspot.com/2011/12/mvvmmvvm-dialog-behavior.html)
* ちょっと Trigger まで頭回ってません → [MVVMパターンでViewModelからViewを操作したい - the sea of fertility](http://ugaya40.net/wpf/mvvm_viewmodel_to_vew.html)
* register/send を持ってる Messenger を View と ViewModel が使うのはちょっと違和感。View には register だけ、ViewModel には send のみを公開する interface を公開すべきかな。
