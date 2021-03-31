---
title: "Hugo Install & Deploy"
date: 2020-12-22T12:11:23+09:00
draft: true
weight: 70
---

## Install Hugo
***
```
$ wget https://github.com/gohugoio/hugo/releases/download/v0.82.0/hugo_0.82.0_Linux-64bit.deb
$ sudo apt install ./hugo_0.82.0_Linux-64bit.deb
$ hugo version
```
```
$ hugo new site workshop
$ cd workshop
$ git init
$ git submodule add https://github.com/matcornic/hugo-theme-learn.git themes/learn
$ echo theme = \"learn\" >> config.toml
$ cat config.toml
$ hugo new posts/my-first-post.md
$ hugo server --bind 192.168.100.136 -p 30000
```
IPアドレス:30000でアクセス可能に

/static/imagesと/contentをコピーすればOK

画像と


