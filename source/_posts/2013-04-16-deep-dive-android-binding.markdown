---
layout: post
title: "Android-Binding の Binding の実装を深掘りしてみる"
date: 2013-04-16 21:28
comments: true
categories: [Android]
---
完全に自分用メモなので乱文です。
<!--more-->
## MVVM とは

Model-View-ViewModel なアーキテクチャ。
View(画面)は、ViewModel のプロパティ(Android-binding では IObservable)の変更を検知し、コントロール(Android ではウィジェット)の内容を置き換える。(OneWay の場合)

TowWay の場合、ユーザーの操作によってコントロールの値が変化した時、ViewModel のプロパティにその変更を適用する。

超断片的なので、詳しくはおググりください。

例：
EditText の Text と、ViewModel の Address プロパティが関連付け(バインド)されていた場合、

* ViewModel.Address がプログラム処理によって変更された時に、EditText.Text が自動的に変更されるのが OneWay。
* 上記に加え、EditText にユーザーが文字を入力した時に、ViewModel.Address にその値が適用されるのが、TwoWay。

## Android-Binding とは
Android で MVVM できるライブラリ。
* [Android-Binding](https://code.google.com/p/android-binding/)

## 何が知りたいのか？
TwoWay の時、EditText にユーザーが文字列を入力して、ViewModel.Address に新しい値を設定する。すると、その変更をまた View が検知して、EditText の値を書き換える処理が動いてしまう(OneWay 側の動きをする)のではないか？

通常の文字列などでは、EditText に同じ値が再設定されるだけなので問題なさげだが、それでも意図しないUI更新は気持ち悪い。

## 調べてみる

Android-Binding では、この辺りを上手く処理しているように見えるので、実装を見てみる。

Android-Binding で、ViewModel 側のプロパティを "Property"、それをバインドする View 側の定義を "Attribute" という。というか勝手にそう呼ぶことにする。(実体はどちらも IObservable の実装だが、この辺を説明しだすと混乱するので割愛)
前述の例だと、「EditText の Text」が "Attribute"、ViewModel.Address が "Property" になる。

1. Attribute に Property がバインドされると、``Attribute#onBind`` が呼び出され、この中で ``Attribute#Bridge`` クラス(これは Observer である)が生成され、Bridge が Property の変更を監視する。と同時に ``this.subscribe(mBridge)`` にて TextViewAttribute 自身の変更も監視する。[[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/Attribute.java#L120)

2. [TextViewAttribute]ユーザーが EditText の Text を変更すると、``TextViewAttribute#onTextChanged`` が呼び出され、さらに ``notifyChanged`` が呼び出される。[[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/viewAttributes/textView/TextViewAttribute.java#L68)

3. [TextViewAttribute]notifyChanged() は Attribute の基底クラスである ``Observable#notifyChanged()`` が呼び出され、続いてオーバーロードである ``notifyChanged(initiators)`` が呼ばれる。 [[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/Observable.java#L66)

4. [TextViewAttribute]``initiators`` とは、.NET イベントの sender のようなもので、イベント(コールバック)発生元のオブジェクトを示す。ただし、``Observable#notifyChanged()`` の時点では、initiators はまだ空っぽ。

5. [TextViewAttribute]``notifyChanged(Collection<Object> initiators)`` にて、initiators に ``this`` つまり TextViewAttribute 自身を追加して、監視している Observer の ``onPropertyChanged`` を呼び出す。[[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/Observable.java#L55)

6. [TextViewAttribute]TextViewAttribute の監視者は ``Attribute#Bridge`` クラスであり、``Bridge#onPropertyChanged(prop, initiators)`` が呼び出される。この時点で、``prop`` は TextViewAttribute 自身、initiators にも TextViewAttribute 自身が格納されている。[[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/Attribute.java#L125)

7. [TextViewAttribute]``Bridge#onPropertyChanged`` の最初の if文 ``if (prop==mAttribute){`` が true となり、``mBindedObservable._setObject(prop.get(), initiators);`` が呼び出される。``mAttribute`` は TextViewAttribute 自身であり、``mBindedObservable`` はバインドした ViewModel の Property である。[[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/Attribute.java#L127)

8. [Property]呼び出されたのは、ViewModel の Property(Observable)、つまり ``Observable#_setObject(newValue, initiators)`` である。initiators には依然として TextViewAttbute が1件格納されている。ここから ``Observable#set(newValue, initiators)`` が呼び出される。[[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/Observable.java#L82)

9. [Property]``Observable#set(newValue, initiators)`` では、``doSetValue`` で、自身(ViewModel の Property)の値を書き換え、その後、initiators に this、つまり ViewModel の Property を追加して ``notifyChanged`` を呼び出す。この時点で initiators は 2件(TextViewAttribute と ViewModelのProperty) である。[[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/Observable.java#L74)

10. [Property]``notifyChanged(Collection<Object> initiators)`` にて、initiators に更に this を追加して(この処理に意味があるのかは謎)、監視者である Observer の ``onPropertyChanged(this, initiators)`` を呼び出す。この時点で initiators は 3件(TextViewAttribute と ViewModelのProperty と ViewModelのProperty) [[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/Observable.java#L55)

11. [TextViewAttribute]再び Bridge#onPropertyChanged(prop, initiators) が呼び出される。ここでは prop は ViewModelのProperty である。これにより、最初の if文 は ``false`` となる。[[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/Attribute.java#L130)

12. [TextViewAttribute]次の if文 ``else if (prop==mBindedObservable){`` は ``true`` となるが、更にその次の if文 ``if (initiators.contains(Attribute.this))`` は、initiators に TextViewAttribute が含まれている為 false となり、処理は終わる。これにより、次の行にある ``mAttribute._setObject(prop.get(), initiators);`` は呼び出されない、つまり、EditText の Text に値が再設定されることはない。[[github]](https://github.com/gueei/AndroidBinding/blob/master/Core/AndroidBinding/src/gueei/binding/Attribute.java#L131)

## まとめ
* Binder の実装は、Bridge が EditText と ViewModelのProperty 両方の変更を監視し、また initiators という仕組みをうまく使うことで、EdtiText が起こした Property の変更は、Attribute が検知しないように工夫されていた。

Android-Binding の仕組みはなんとかわかったー。.NET の XAML はどうやってんのかな？
