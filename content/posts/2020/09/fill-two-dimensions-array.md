---
title: "JavaScriptでArray.fill()を使用して二次元配列を作る時の注意点"
date: 2020-09-30T17:15:10+09:00
draft: false
tags: ["TypeScript", "JavaScript"]
---

この記事について

## この記事について

この記事では、JavaScriptでArray.fill()を使用して二次元配列を作る際の注意点について書いていきます。

自分がコードを書いた時に出会った問題を解決する時に知った事を残しておきます。

## 注意点について

簡単に言うとArray.fill()に配列(オブジェクト)を渡すと、そのオブジェクトへの参照がコピーされ配列に参照が格納されるという点です。

つまりオブジェクト(配列)の一箇所変更をしたら他のオブジェクト(配列)も釣られて変更されてしまう可能性があるということです。

例えばあなたが三目並べを作っていて、ボードを表す二次元配列を

```typescript
const board = Array(3).fill(['', '', '']);
```

と作った場合、

```
[
	['', '', ''],
	['', '', ''],
	['', '', '']
]
```

という二次元配列が出来上がります。

そしてバツ(✗)を表すプレイヤーが真ん中に入力したのでボードの状態を変更したとしましょう。

すると

```
[
	['', '✗', ''],
	['', '✗', ''],
	['', '✗', '']
]
```

という状態になってしまいます。



これは３つの配列すべてが同じ参照(fill()に渡した[' ', ' ', ' '])を使用しているため、このようになります。

それぞれの配列データがfill()に渡された配列をコピーし独立しているわけではありません。

## どういう問題に遭遇したか

Reactでなんか作りたいなと思ってリバーシを作ろうとしてる時に、ボードの初期化処理で上の挙動に悩まされたので今回は調べてみました。

同じではないですがこんな感じで

```typescript
function createInitialBoard() {
    const rowLength = 8;
    const columnLength = 8;
    //まずは何も置かれてないボードを二次元配列で表現する
    const board = Array(columnLength).fill(Array(rowLength).fill('NONE'));
    //試しに適当な場所に石を置く
    board[3][3] = 'BLACK';
    //以下省略
}
```

ボードを表現して初期石を配置するとReactは最終的に

![Screenshot from 2020-09-30 17-50-17](https://user-images.githubusercontent.com/37544784/94664467-e1bfc400-0345-11eb-8c48-0b34002b8b61.png)

とレンダリングしていました。そうですね、上にあるfillの仕様を知らなかったために全部変更されてますね。

## 解決策

```typescript
const board = Array(3).fill().map(() => ['', '', '']);
```

とmap()を利用しましょう

fill()に引数を指定せず使用した場合、undefinedで埋めてくれます。その上でmap()が新たな配列を返すのを利用しそれぞれ独立した二次元配列を構築します。



最後まで読んでくれてありがとうございました。

## 出典

[MDN - Array.prototype.fill()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/fill#Description)

