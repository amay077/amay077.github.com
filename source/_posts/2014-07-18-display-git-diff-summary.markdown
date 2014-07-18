---
layout: post
title: "git で過去との差分のサマリーを表示する"
date: 2014-07-16 00:00
comments: true
categories: [git]
---
「現在のソースコードは、過去のバージョンからどのくらい修正したか？」を知りたい時に使います。
<!--more-->

## コマンド

```
git diff --stat <コミットID>

例）git diff --stat c7378a8
```

## 結果

```
 amay077lib                                         |   0
 .../android/fujiphoto/logic/PhotoItemLogic.java    |   1 -
 .../android/fujiphoto/viewmodel/YmapBinder.java    |  24 +-
 graphics/app_icon.svg                              | 436 +++++++++++++++++++++
 graphics/camera.svg                                |  97 +++++
 graphics/circle_small.svg                          |  87 ++++
 graphics/fuji.svg                                  | 118 ++++++
 graphics/loading.svg                               | 142 +++++++
 graphics/pegman.svg                                | 176 +++++++++
 graphics/photo_loading.svg                         | 142 +++++++
 graphics/photo_loading_large.svg                   |  92 +++++

 11 files changed, 1301 insertions(+), 14 deletions(-)
```
