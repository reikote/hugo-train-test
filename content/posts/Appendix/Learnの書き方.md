---
title: "Markdown記法とLearnの作法"
date: 2020-12-22T11:34:44+09:00
draft: false
weight: 60
---

### Markdown記法

こちらを参考にしましょう
- [Wikipedia](https://ja.wikipedia.org/wiki/Markdown)
- [Qiita](http://qiita.com/Qiita/items/c686397e4a0f4f11683d)
- [日本語Markdownユーザー会](http://www.markdown.jp/what-is-markdown/)

# 様々な書き方

# h1
## h2
### h3
#### h4
##### h5
###### h6

### コメント
<!--
これはコメント
-->

### 水平線
<!-- 全部一緒 -->
___
---
***

### 太字
**ABC**
_abc_
~~ABC~~

### ブロック表示
> こうやってやるんだよ
>> ネストもできる

### 箇条書き
<!-- タブで段差をずらす -->
+ aaa
+ bbb
+ ccc
	- ddd
	- eee
	- fff
	+ aaa
	* bbb

### 順序書き
1. aaa
2. bbb
3. ccc
5. ddd
4. eee

### インライン
aaa`aaa`bbb

### よくわからん
// aaa aa
bbbb
cccc

### コードの書き方
```
aaaa
bbbb
dfaefa
fdafa
```

### 表の書き方

| Command | Description |
| ------ | ----------- |
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |

右寄せ
| Option | Description |
| ------:| -----------:|
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |

### リンク
[Assemble](http://assemble.io)

TIPSを表示
[Upstage](https://github.com/upstage/ "押してくれ！！！")

### ジャンプする
  * [インラインのとこ](#インライン)
  * [コードのとこ](#コードの書き方)
  

### 画像
![画像](https://octodex.github.com/images/minion.png?height=400px)
後から画像差し込み可能
![画像][id1]
ローカル画像は、staticフォルダに格納すれば使用できる
![画像](/images/vmware_login.PNG?height=400px)
widthとheight両方も可能
![画像](https://octodex.github.com/images/minion.png?height=50px&width=300px)

[id1]: https://octodex.github.com/images/dojocat.jpg?width=200px

画像に枠や影をつけることが可能
shadowやborder等
![stormtroopocat](https://octodex.github.com/images/stormtroopocat.jpg?classes=shadow&height=200px)
![stormtroopocat](https://octodex.github.com/images/stormtroopocat.jpg?classes=border,shadow&height=200px)
クリックできない画像にすることも可能
![Minion](https://octodex.github.com/images/minion.png?featherlight=false&height=200px)

### 文字の色変更
わからない

### 通知

{{% notice note %}}
A notice disclaimer
{{% /notice %}}

{{% notice info %}}
An information disclaimer
{{% /notice %}}

{{% notice tip %}}
A tip disclaimer
{{% /notice %}}

{{% notice warning %}}
A warning disclaimer
{{% /notice %}}

### 目次
{{% children depth="3" showhidden="true" %}}

