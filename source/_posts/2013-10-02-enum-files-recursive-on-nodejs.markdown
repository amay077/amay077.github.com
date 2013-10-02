---
layout: post
title: "ディレクトリを再帰的にたどってファイル一覧を出力する"
date: 2013-10-02 19:20
comments: true
categories: [Node.js, JavaScript]
---
[@_shimizu](https://twitter.com/_shimizu) さんの
<!--more-->
* [[node.js]ディレクトリを再帰的にたどってファイル一覧をJSONとして出力する | GUNMA GIS GEEK](http://shimz.me/blog/node-js/2944)

の派生品です。ファイルパスを出力するだけにしました。

```js enumFilesRecursive.js
var fs = require("fs")
    , path = require("path")
    , dir = process.argv[2] || '.'; //引数が無いときはカレントディレクトリを対象とする

var walk = function(p, fileCallback, errCallback) {

	fs.readdir(p, function(err, files) {
		if (err) {
			errCallback(err);
			return;
		}

		files.forEach(function(f) {
			var fp = path.join(p, f); // to full-path
			if(fs.statSync(fp).isDirectory()) {
				walk(fp, fileCallback); // ディレクトリなら再帰
			} else {
				fileCallback(fp); // ファイルならコールバックで通知
			}
		});
	});
};


// 使う方
walk(dir, function(path) {
	console.log(path); // ファイル１つ受信	
}, function(err) {
	console.log("Receive err:" + err); // エラー受信
});
```