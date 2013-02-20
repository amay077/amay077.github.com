---
layout: post
title: "Mac に Homebrew で Haskell 入れて HelloWorld"
date: 2012-10-20 14:22
comments: true
categories: [Haskell]
---
OS は Mountain Lion、Homebrew は入ってる前提で。

##インストール
「処理系」の GHC と、開発用プラットフォームをインストールする。

```
brew install ghc
```

>==> Downloading http://www.haskell.org/ghc/dist/7.4.2/ghc-7.4.2-x86_64-apple-darwin.tar.bz2
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/ghc/7.4.2
==> make install
==> Caveats
This brew is for GHC only; you might also be interested in haskell-platform.
==> Summary
/usr/local/Cellar/ghc/7.4.2: 6209 files, 842M, built in 18.9 minutes

20分弱かかった。

```
brew install haskell-platform
```
>==> Downloading http://lambda.haskell.org/platform/download/2012.2.0.0/haskell-platform-2012
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/haskell-platform/2012.2.0.0 --enable-unsupported-
==> EXTRA_CONFIGURE_OPTS="--libdir=/usr/local/Cellar/haskell-platform/2012.2.0.0/lib/ghc" ma
==> Caveats
Run `cabal update` to initialize the package list.
>
If you are replacing a previous version of haskell-platform, you may want
to unregister packages belonging to the old version. You can find broken
packages using:
  ghc-pkg check --simple-output
You can uninstall them using:
  ghc-pkg check --simple-output | xargs -n 1 ghc-pkg unregister --force
==> Summary
/usr/local/Cellar/haskell-platform/2012.2.0.0: 869 files, 174M, built in 13.4 minutes

全部で３０分ほど。途中「ほんとに動いてんの？」って状態になったが焦らず放っておいたら終わってた。

##対話式インタプリタを起動する

```
ghci
```
>GHCi, version 7.4.2: http://www.haskell.org/ghc/  :? for help
Loading package ghc-prim ... linking ... done.
Loading package integer-gmp ... linking ... done.
Loading package base ... linking ... done.
Prelude> 

##ハロワ
```
Prelude> putStrLn "Hello, World"
```

>Hello, World

##GHCi の終了
意外と分からなかった。

```
:quit
```

>Leaving GHCi. 

##【未解決】runhaskell で「lexical error (UTF-8 decoding error)」エラー

[10分で学ぶHaskell - HaskellWiki](http://www.haskell.org/haskellwiki/10%E5%88%86%E3%81%A7%E5%AD%A6%E3%81%B6Haskell) にあった REPL じゃなくてソースをビルドして実行する方法でエラーが。

```hs Test.hs
main = do putStrLn "What is 2 + 2?"
          x <- readLn
          if x == 4
              then putStrLn "You're right!"
              else putStrLn "You're wrong!"
```

を Test.hs で保存して、

```
ghc --make Test.hs
```

を実行して、Test, Test.hi, Test.o ファイルが生成される。
で、

```
runhaskell Test
```
とすると

>Test:1:1: lexical error (UTF-8 decoding error)

というエラーが。よく分からないからとりあえず、

```
runghc Test.hs
```
で満足してる。まだ入り口にも立ってない気分。

##参考
* [10分で学ぶHaskell - HaskellWiki](http://www.haskell.org/haskellwiki/10%E5%88%86%E3%81%A7%E5%AD%A6%E3%81%B6Haskell)
* [Haskell再入門 1日目(環境構築) - WebService::Blog->new( user => ’hide_o_55’ )](http://d.hatena.ne.jp/hide_o_55/20110427/1303914801)

