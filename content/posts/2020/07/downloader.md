---
title: "GitHubリポジトリ上の任意のディレクトリをワンライナーでダウンロードする(awkとsvn)"
date: 2020-07-19T08:48:47+09:00
draft: false
tags: ["awk", "Git", "シェルスクリプト"]
---

## 結論

<span style="color: red; ">Apache Subversion が必要です。</span>

```bash
echo "https://github.com/mattn/go-gtk/tree/master/_example/demo
" | awk '{gsub("tree/master", "trunk", $1); print $1}' | xargs svn export
```

ただこれだと使いにくいのでシェルスクリプト化してみました。

```bash
#!/bin/bash

echo "Downloading...."
echo $1 | awk '{gsub("tree/master", "trunk", $1); print $1}' | xargs svn export
```

これを svnd というファイル名で保存して svnd コマンドとして使えるようになったとすると

```bash
svnd https://github.com/mattn/go-gtk/tree/master/_example/demo
```

とコマンドの後ろに URL を貼り付けるだけで使えます。

僕は fish shell を使っているので fish 版も作りました。

```bash
#!/usr/bin/fish

echo "Downloading...."
echo $argv[1] | awk '{gsub("tree/master", "trunk", $1); print $1}' | xargs svn export
```

## 環境

- Linux Ubuntu 18.04
- fish shell or bash
- Apache Subversion

## なぜこれを作ったか

OSS の GitHub リポジトリなんかを眺めてて、よく使用例などを集めた examples ディレクトリが作られている事があります。
example を試すだけなのに大きなリポジトリを Clone してくるのはしんどいので試したい example ディレクトリ単体をダウンロードしたいなーといつも思ってました。

どうやら subversion を使って

```
svn export 任意のGitHubリポジトリのディレクトリURL
```

するとダウンロードできる事を知りました。

ただ、この方法だとディレクトリ URL を少し加工しなければなりません。

例えばディレクトリ URL が

```
https://github.com/mattn/go-gtk/tree/master/_example/demo
```

だとすると

```
https://github.com/mattn/go-gtk/trunk/_example/demo
```

に変更してから先程の svn コマンドを打たなければなりません。

どこが違うかと言うと "tree/master"が消えて、その部分が "trunk"　に置き換わっています。

これを毎回手作業で弄るのはダルいので awk を使い自分用にコマンド化しました。

やってることはただの置換です。

## GitHub リポジトリ上の任意のファイルをダウンロードするツールも作りました

これはついでなんですが、Go で GitHub リポジトリ上の任意のファイルをダウンロードする CLI アプリも作りました。

これも作った理由は先程とほぼ同じです。

ダウンロードしたいシングルファイルが

```
https://github.com/mattn/go-gtk/blob/master/_example/demo/demo.go
```

だった場合

```
https://github.com/mattn/go-gtk/raw/master/_example/demo/demo.go
```

と、 "blob"　部分を "raw"に変換してから

```bash
wget https://github.com/mattn/go-gtk/raw/master/_example/demo/demo.go
```

しなければなりません。

こっちは Go を学びたてとあって Go で作りました。少しだけ機能追加してます。

Linux 上でしか動かしてませんが、Windows と Mac 向けにもビルドしてあるので多分動きます。

リポジトリ
https://github.com/KoutaKawase/wakana

バイナリダウンロード https://github.com/KoutaKawase/wakana/releases/tag/v1.0.0

ダウンロードするファイルをリネームしたりダウンロード先のディレクトリを指定できます。

## お礼

ここまで読んでくださってありがとうございます！
良かったら[Twitter](https://twitter.com/omochizou)もやっているので、フォローしていただけるとすごく喜びます 👍
