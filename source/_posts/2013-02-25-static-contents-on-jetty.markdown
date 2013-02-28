---
layout: post
title: "jetty で静的コンテンツを有効にして起動するまで"
date: 2013-02-25 19:33
comments: true
categories: [jetty]
---
jetty で静的コンテンツを有効にして起動するまで

index.html とかホストしたいだけなんですけど、意外に苦労したのでメモ。

<!-- more -->

## jetty のダウンロード
[Jetty Ecripse Downloads](http://download.eclipse.org/jetty/) から stable-8 をダウンロード。(9 がまだ RC だったおので、念のため)

## 適当なディレクトリに展開
```
~/dev/sdks/jetty-8.1.9
```
にしてみました。

## とりあえず起動してみる

```
$ cd ~/dev/sdks/jetty-8.1.9
$ java -jar start.jar
```

http://localhost:8080 にアクセス。

動いた、ふむふむ。

## 普通の HTML とかをぶっこむディレクトリを作る

'htdocs' にしました。

```
$ cd ~/dev/sdks/jetty-8.1.9
$ mkdir htdocs
```

適当な HTML を htdocs に入れる。

``` html ~/dev/sdks/jetty-8.1.9/htdocs/index.html
<!DOCTYPE html>
<html lang="ja">
<body>
	<H1>Hello World!</H1>
</body>
</html>
```

## jetty の設定ファイルで、静的コンテンツを有効にする Handler とかいうやつを設定する

``etc/jetty.xml`` をコピーして、``etc/jetty_static.xml`` とか適当な名前にする。

handler の設定のところに ResourceHandler のブロックを入れる

``` xml jetty_static.xml
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure.dtd">

<!-- =============================================================== -->
<!-- Configure the Jetty Server                                      -->
<!--                                                                 -->
<!-- Documentation of this file format can be found at:              -->
<!-- http://wiki.eclipse.org/Jetty/Reference/jetty.xml_syntax        -->
<!--                                                                 -->
<!-- Additional configuration files are available in $JETTY_HOME/etc -->
<!-- and can be mixed in.  For example:                              -->
<!--   java -jar start.jar etc/jetty-ssl.xml                         -->
<!--                                                                 -->
<!-- See start.ini file for the default configuraton files           -->
<!-- =============================================================== -->


<Configure id="Server" class="org.eclipse.jetty.server.Server">

    <!-- =========================================================== -->
    <!-- Server Thread Pool                                          -->
    <!-- =========================================================== -->
    <Set name="ThreadPool">
      <!-- Default queued blocking threadpool -->
      <New class="org.eclipse.jetty.util.thread.QueuedThreadPool">
        <Set name="minThreads">10</Set>
        <Set name="maxThreads">200</Set>
        <Set name="detailedDump">false</Set>
      </New>
    </Set>

    <!-- =========================================================== -->
    <!-- Set connectors                                              -->
    <!-- =========================================================== -->

    <Call name="addConnector">
      <Arg>
          <New class="org.eclipse.jetty.server.nio.SelectChannelConnector">
            <Set name="host"><Property name="jetty.host" /></Set>
            <Set name="port"><Property name="jetty.port" default="8080"/></Set>
            <Set name="maxIdleTime">300000</Set>
            <Set name="Acceptors">2</Set>
            <Set name="statsOn">false</Set>
            <Set name="confidentialPort">8443</Set>
	    <Set name="lowResourcesConnections">20000</Set>
	    <Set name="lowResourcesMaxIdleTime">5000</Set>
          </New>
      </Arg>
    </Call>

    <!-- =========================================================== -->
    <!-- Set handler Collection Structure                            --> 
    <!-- =========================================================== -->
    <Set name="handler">
      <New id="Handlers" class="org.eclipse.jetty.server.handler.HandlerCollection">
        <Set name="handlers">
         <Array type="org.eclipse.jetty.server.Handler">

           <!-- ここから -->
           <Item>
            <New class="org.eclipse.jetty.server.handler.ResourceHandler">
             <Set name="resourceBase"><Property name="files.base" default="./htdocs"/></Set>
            </New>
           </Item>
           <!-- ここまで -->

			<!-- よくわからないのでコメントアウトしてみた
           <Item>
             <New id="Contexts" class="org.eclipse.jetty.server.handler.ContextHandlerCollection"/>
           </Item>
           <Item>
             <New id="DefaultHandler" class="org.eclipse.jetty.server.handler.DefaultHandler"/>
           </Item> -->
         </Array>
        </Set>
      </New>
    </Set>

    <!-- =========================================================== -->
    <!-- extra options                                               -->
    <!-- =========================================================== -->
    <Set name="stopAtShutdown">true</Set>
    <Set name="sendServerVersion">true</Set>
    <Set name="sendDateHeader">true</Set>
    <Set name="gracefulShutdown">1000</Set>
    <Set name="dumpAfterStart">false</Set>
    <Set name="dumpBeforeStop">false</Set>

</Configure>
```

## 設定ファイルを指定して起動してみる

```
$ cd ~/dev/sdks/jetty-8.1.9
$ java -jar start.jar --ini etc/jetty_static.xml
```

--ini は不要な気もするのだけど、付けないとエラーになっちゃった。

これで、 http://localhost:8080 にアクセスすると、さっきつくった Hello World が表示されるはず。


