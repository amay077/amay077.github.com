---
layout: post
title: "haxenode でエラー window is not defined"
date: 2012-10-03 22:06
comments: true
categories: [haxe, Node.js, haxenode]
---
[HaxeでつくるWebアプリ開発(node.js + Express) | 深追い Fukaoi.org](http://blog.fukaoi.org/2012/06/19/haxe_nodejs_express) を参考に、[haxenode](http://haxenode.org/)を使ってみたときにハマったこと。

<!-- more -->

```as rootnode.hx
import js.Node;

class RootNode {
	static function main() {
	 var server = Node.http.createServer( function(
	      req:NodeHttpServerReq, res:NodeHttpServerResp){
	        res.setHeader("Content-Type","text/plain");
	        res.writeHead(200);
	        res.end(Hoge.print());
	      }
	    );
	 
	    server.listen(1337,"localhost");
	    trace( 'Server running at http://127.0.0.1:1337/' );
   	}	
}

@:keep
@:expose
class Hoge {
	public static function print():String {
		return "Hello World.\n";
	}
}
```

こんな感じでサンプル作って、

	node rootnode.js

を実行したところ、


>/.../bin/haxenode.js:53
	var o = window;
	        ^
ReferenceError: window is not defined
    at $hxExpose (/.../bin/haxenode.js:53:10)
    at /.../bin/haxenode.js:13:1
    at Object.<anonymous> (/.../bin/haxenode.js:62:3)
    at Module._compile (module.js:449:26)
    at Object.Module._extensions..js (module.js:467:10)
    at Module.load (module.js:356:32)
    at Function.Module._load (module.js:312:12)
    at Module.runMain (module.js:492:10)
    at process.startup.processNextTick.process._tickCallback (node.js:244:9)

こんなエラーが出ました。

いろいろ試したところ、Hoge クラスに ```@:keep``` と ```@:expose``` をどちらも指定するとダメみたい。どちらか片方だけなら正常。
[--dead-code-elimination](http://haxe.org/doc/compiler) をあきらめて、minify とかすればいいのかな。
``@:keep`` と ``@:expose`` は意味が被る、というか expose は keep を包括するような感じなので、@:expose だけでよさそうな気もします。

##参考
* [Tips and Tricks - Haxe](http://haxe.org/manual/tips_and_tricks)