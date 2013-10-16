---
layout: post
title: "YAML Front-matter とかいうやつ"
date: 2013-10-16 16:08
comments: true
categories: [Markdown, Qiita]
---
Octopress でもブログ書いてるんだけど、markdown のヘッダに、
<!--more-->
```
---
layout: post
title: "metersToEquatorPixels を Google Maps Android API v2 で"
date: 2013-10-09 00:21
comments: true
categories: [Android, Java, Geo, GoogleMapsAPI]
---
```

こんな風にメタ情報を書くルールになっている。

この書き方、 Front-matter というそうで。

* [Front-matter - jekyllrb](http://jekyllrb.com/docs/frontmatter/)

んで、github は、これをいい感じに整形して表示してくれる。

* [amay077.github.com/source/_posts/2013-10-09-meterstoequatorpixels-in-gmap-v2.markdown](https://github.com/amay077/amay077.github.com/blob/source/source/_posts/2013-10-09-meterstoequatorpixels-in-gmap-v2.markdown)

@Qiita とか ＠Kobito でもこれに対応してくれると、私が嬉しい。

Kobito で「書き出し」た .md ファイルのヘッダが Front-matter になってたり、逆に Front-matter を読み込んでくれるといいな。