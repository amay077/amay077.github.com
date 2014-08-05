---
layout: post
title: "いまさら Docker やってみた"
date: 2014-08-05 21:39:53 +0900
comments: true
categories: [Docker, Linux, Ubuntu]
---
いまさら感満載なんですが、Docker やってみました。
<!--more-->

ホストPCはMacです。

## Docker のインストール

Macなので事前に VirtualBox を入れて置く必要があります。

あとは、

* [はじめてのDocker on Mac OS X ｜ Developers.IO](http://dev.classmethod.jp/tool/docker/getting-started-docker-on-osx/)

を参考に。

Docker と boot2docker のバージョンはこんな感じでした。

```sh
$ docker -v
Docker version 1.1.2, build d84a070

$ boot2docker version
Client version: v1.1.2
Git commit: a229ac1
```

## boot2docker 経由で Docker の起動

``boot2docker up`` で Docker (というか Tiny Core Linux らしい)を起動。

```sh
$ boot2docker up
2014/08/05 21:47:19 Waiting for VM to be started...
.......
2014/08/05 21:47:41 Started.
2014/08/05 21:47:41 Your DOCKER_HOST env variable is already set correctly.
```

``set:export DOCKER_HOST=`` とか出たらホスト（Mac） .bash_profile にそのまま追記します。（上はすでに追加済なので ``already`` と出てます）

## boot2docker から Docker に接続

``boot2docker ssh`` で Docker に接続。

```sh
$ boot2docker ssh
                        ##        .
                  ## ## ##       ==
               ## ## ## ##      ===
           /""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
           \______ o          __/
             \    \        __/
              \____\______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
boot2docker: 1.1.2
             master : 740106c - Thu Jul 24 03:24:10 UTC 2014
docker@boot2docker:~$ 
```

## Docker 内で ubuntu を起動する

``docker run`` で docker のイメージを（無ければダウンロードして）起動し、後続のコマンドを実行して終わる。
下記の場合 bash を起動しているので、``exit`` すると終わる。

```sh
docker@boot2docker:~$ sudo docker run -i -t ubuntu /bin/bash ＜--docker内
Unable to find image 'ubuntu' locally
Pulling repository ubuntu
ba5877dc9bec: Download complete 
511136ea3c5a: Download complete 
9bad880da3d2: Download complete 
25f11f5fb0cb: Download complete 
ebc34468f71d: Download complete 
2318d26665ef: Download complete 
root@c8b319b2b306:/# ＜--ubuntu に入った
```

## アタッチとかデタッチとか

CTRL+p → CTRL+q と押すと、Ubuntu からデタッチして docker に戻る。（ubuntu は終わらない）

``docker ps`` とすると起動している ubuntu の一覧が見える。

```sh
ocker@boot2docker:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c8b319b2b306        ubuntu:latest       /bin/bash           5 minutes ago       Up 5 minutes                            prickly_tesla       
```

もう一度 ``sudo docker run -i -t ubuntu /bin/bash`` とすると、ubuntu がもう１個起動する。
CTRL+p → CTRL+q でデタッチして、``docker ps`` すると、２つになってるのが分かる。

```sh
docker@boot2docker:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4744cf404fda        ubuntu:latest       /bin/bash           4 seconds ago       Up 4 seconds                            distracted_einstein   
c8b319b2b306        ubuntu:latest       /bin/bash           8 minutes ago       Up 8 minutes                            prickly_tesla         
```

起動中の ubuntu に入るには、``docker attach c8b319b2b306`` などとする。この場合 bash が実行中なので ubuntu のコンソールになる。

```sh
docker@boot2docker:~$ docker attach c8b319b2b306

root@c8b319b2b306:/# 
```

## 起動しているOSの削除

``docker attach`` して ``exit`` するか、docker側から ``docker kill c8b319b2b306`` などとする。

``docker ps`` すると、削除されたのが分かる。

```sh
docker@boot2docker:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4744cf404fda        ubuntu:latest       /bin/bash           3 minutes ago       Up 3 minutes                            distracted_einstein   
```

## OSの状態の保存と復元

docker から起動した ubuntu は揮発性なので、終了すると状態が消えてしまう。
が、ほんとは消えてなくて、終了した後で、履歴からイメージを作成することができる。

まず、ubuntu にて適当なファイルを作成。

```sh
root@4744cf404fda:/# echo 'Hello' >> mytext

root@4744cf404fda:/# ls
bin   dev  home  lib64	mnt	opt   root  sbin  sys  usr
boot  etc  lib	 media	mytext	proc  run   srv   tmp  var
```

確かに ``mytext`` が作成されているのを確認したら、``exit`` で終了、docker に戻る。

``sudo docker run -i -t ubuntu /bin/bash`` で ubuntu を起動し、``ls`` で、内容を確認。

```sh
root@3d15ee0cc161:/# ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
```

``mytext`` は消えている。
確認後 ``exit`` で終了。

``docker ps -a`` で履歴も含めて状態を見る。

```sh
docker@boot2docker:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
3d15ee0cc161        ubuntu:latest       /bin/bash           19 seconds ago      Exited (0) 12 seconds ago                       dreamy_hawking        
4744cf404fda        ubuntu:latest       /bin/bash           8 minutes ago       Exited (0) 30 seconds ago                       distracted_einstein   
16d24570a714        ubuntu:latest       /bin/bash           12 minutes ago      Exited (0) 12 minutes ago                       focused_wilson        
```

``mytext`` を保存したのは 4744cf404fda の ubuntu なので、これを保存する。
``docker commit`` で git ライクにコミットすると保存される。

```sh
docker@boot2docker:~$ docker commit -m "Add mytext" 4744cf404fda amay077/mytext_container

467f6424ae4a7b813f51356a019ef6ee2467fe2f1f52d8ea7a2e32ddc0b63edd
```

``docker images`` を実行して保存されているイメージのリストを見る。

```sh
REPOSITORY                 TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
amay077/mytext_container   latest              467f6424ae4a        50 seconds ago      192.7 MB
ubuntu                     latest              ba5877dc9bec        2 weeks ago         192.7 MB
d
```

``amay077/mytext_container`` が確かに保存されてる。

そのイメージを使ってインスタンスを起動する。

```sh
docker@boot2docker:~$ sudo docker run -i -t amay077/mytext_container /bin/bash

root@0f6e755990ff:/# ls
bin   dev  home  lib64	mnt	opt   root  sbin  sys  usr
boot  etc  lib	 media	mytext	proc  run   srv   tmp  var

root@0f6e755990ff:/# cat mytext
Hello
```

確かに mytext が保存された状態になっている。

## コンテナとイメージの削除

いきなり「コンテナ」ってｗ

イメージから起動した「モノ」をコンテナというらしい。
起動して終了したもの（``docker ps -a`` で見られるもの）もコンテナというらしい。

OSを終了してから、``docker rm 3d15ee0cc161`` などで個別に削除してもよいが、面倒なので、
``docker rm `docker ps -a -q`` とすると、起動してないコンテナを一括削除できる。

```sh
docker@boot2docker:~$ docker rm `docker ps -a -q`
3d15ee0cc161
4744cf404fda
16d24570a714
57ee3aa4a7d2
c8b319b2b306

docker@boot2docker:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

続いて、イメージの削除は ``docker rmi amay077/mytext_container`` などで行う。

```sh
docker@boot2docker:~$ docker rmi amay077/mytext_container

docker@boot2docker:~$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              latest              ba5877dc9bec        2 weeks ago         192.7 MB
```

ubuntu だけになった。
``docker rmi ubuntu`` もするときれいサッパリ。

今日はここまで。


## 参考サイト

* [はじめてのDocker on Mac OS X ｜ Developers.IO](http://dev.classmethod.jp/tool/docker/getting-started-docker-on-osx/)
* [仮想環境構築に docker を使う - apatheia.info](http://apatheia.info/blog/2013/06/17/docker/)
* [Dockerで不要になったコンテナやイメージを削除する - @znz blog](http://blog.n-z.jp/blog/2013-12-24-docker-rm.html)


