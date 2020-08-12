---
title: "よく分からなかったのでCommonJSとTypeScriptのES Modulesinteropについて調べてみた"
date: 2020-08-21T05:55:05+09:00
draft: false
tags: ["TypeScript", "JavaScript"]
---

TypeScriptでExpressをちょびちょび弄ってて、Expressをimportする際esModuleinterop関連のエラーに引っかかったので、ついでによく分からなかったCommonJSやesModuleinteropフラグについて自分なりに調べてまとめておきます。

具体的にはtsconfig.jsonを作成しないで

```typescript
import express from 'express';
```

としたら

```typescript
node_modules/@types/express/index.d.ts:116:1 
    116 export = e;

This module is declared with using 'export =', and can only be used with a default import when using the 'esModuleInterop' flag.
// このモジュールは 'export =' で宣言されており、'esModuleInterop' フラグを使用している場合にのみデフォルトのインポートで使用することができます。
```

と出た時の話です。

端的に言うと、これを解決するにはメッセージ通りにtsconfig.jsonでesModuleInteropフラグを有効にする以外に

```typescript
import express = require('express');
```

という記法で書いても解決できます。

TypeScriptでは、"*export = *"の記法でエクスポートされたものはTypeScript固有の

```typescript
 import module = require("module")
```

といった記法でimportする必要があるからです。

(参考: https://www.typescriptlang.org/docs/handbook/modules.html#export--and-import--require)

今回はその方法ではなく、もう一つのesModuleInteropフラグについてとCommonJSについて書いていきます。(CommonJSについて書くのはExpressがCommonJSモジュールで構成されてるからです)

## CommonJSとは

一言で言うと、*ブラウザ外におけるJavaScriptのモジュールシステムの仕様を策定することを目的としたプロジェクト*です。

ブラウザ外というと、サーバーサイドにおけるNode.jsとかですね。

### module.exports と exportsの違い

最初、Expressのコードを読んでいてCommonJSにおけるこの２つの違いがよく分かりませんでした。

大事なのはこの２つは根本的には同じということです。

どういう事かと言うと、内部ではこの２つは

```javascript
exports = module.exports = {}
```

という風に同じ空のオブジェクトを指しています。

従って

```javascript
exports.hoge = "hoge";
module.exports.fuga = "fuga";
```

とした場合、requireした側から見れば

```javascript
{ hoge: 'hoge', fuga: 'fuga' }
```

という一つのオブジェクトとして認識されます。

Node.jsの公式ドキュメントでexportsがmodule.exportsのショートハンド版だと言われてるのはこのためです。

(出典: https://nodejs.org/api/modules.html#modules_exports_shortcut)

これらの使い分けとして

module.exportsはrequireして得られる値をそのままコンストラクタや関数として使いたい時に使いましょう。

例えば

```javascript
//Game.js
module.exports = class Game {
    constructor() {...}
}
```

みたいなクラスをmodule.exportsしたとすると、

```javascript
const Game = require('Game');

const game = new Game();
```

といったように、そのままnewして使えたりできます。

これがexportsでエクスポートしていたならば

```javascript
//Game.js
exports.Game = class Game {...}
```

```javascript
const game = require('Game');

const game = new game.Game();
```

という書き方になり面倒です。

ただしES6では

```javascript
const { Game } = require('Game');

const game = new Game();
```

といったオブジェクトの分割代入という記法も使えます。(参考: https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Object_destructuring)



対してexportsは、複数の関数やプロパティなどをエクスポートしたい時、requireした際得られる値をオブジェクトとして得たいときに使います。

```javascript
//calc.js
exports.add = (num1, num2) => {...}

exports.sub = (num1, num2) => {...}
```

```javascript
//main.js
const calc = require('./calc');

console.log(calc); // { add: [Function], sub: [Function] }

calc.add(1, 1); //2
```

みたいな感じです。

## esModuleinteropとは

tsconfig.jsonのcompiler optionsには *esModuleinterop*というフラグがあります。

これはCommonJSとESモジュール間との相互運用を可能にするためのフラグです。

なぜこんなフラグが存在するのかと言うと、ES ModulesとCommonJSとの互換性に規約がないためです。

TypeScriptはESModulesに準拠していて、CommonJSモジュールを読み込むのに*import React from 'react'*という構文をサポートするにはCommonJSモジュール側で *exports.default* というプロパティがエクスポートされている必要が有ります。　わざわざそんなものを書いてるものはレアなので、esModuleinteropがCommonJS*側のモジュールに自動でdefault*を付け加えています。

実例を見てもらうと、

```javascript
//server.ts
import express from 'express';

const app = express();
```

というコードがあるとしてesModuleInteropを使いTypeScriptはこうコンパイルします。

```typescript
//server.js
"use strict";
var __importDefault = (this && this.__importDefault) || function (mod) {
    return (mod && mod.__esModule) ? mod : { "default": mod };
};
Object.defineProperty(exports, "__esModule", { value: true });
var express_1 = __importDefault(require("express"));
var app = express_1.default();
```

２行目でrequireして得たオブジェクトにdefaultプロパティを追加しているのが伺えます。

esModuleInteropはコード生成時にこうしてCommonJSとES Moduleの相互運用性を担保しています。

つまりランタイムレベルで関係のあるCommonJSとES Modulesに関するフラグだという事です。

対して、TypeScriptのcompiler optionsには *allowSyntheticDefaultImports* というフラグが存在します。

このフラグはesModuleInteropフラグが有効であれば自動でオンとなっています。

## allowSyntheticDefaultImportsとは

こちらもesModuleInteropフラグ同様、CommonJSとES Modulesとの相互運用を可能にするためフラグです。

じゃあ何が違うのかと言うと *実行時に影響せず、型チェックのみに影響する* という点です。

どういう事かと言うとそのまま *import React from 'react'* のように書くとコンパイルエラーを吐きますが、allowSyntheticDefaultImportsを付けることでとりあえずコンパイルエラーは出ずコンパイルを通るようになります。

これをつけることで *import React from 'react'* という書き方をコンパイルエラーなしでとりあえずできます。

ですがあくまでコンパイル時のみに影響するものなので、コード生成には何の影響も及ぼさずコンパイルされたコードは動かないままでしょう。

esModuleInteropとallowSyntheticDefaultImportsの違いは端的に言うと *実行時に関係するかコンパイル時のみに関係するか*　です。

実験してみましょう。

```TypeScript
//main.ts
import express from 'express';

const app = express()
```

というソースコードを用意します。

```bash
npx tsc main.ts # tscはファイル名を指定してコンパイルするとtsconfigが読み込まれないので、allowSyntheticDefaultImportsはオフになってます
```

とするとコンパイルエラーを吐き出します。

ここで、allowSyntheticDefaultImportsを使うと

```bash
npx tsc --allowSyntheticDefaultImports true main.ts
```

問題なくコンパイルが通るはずです。

コンパイルされた main.js を実行してみましょう。

```bash
node main.js # TypeError: express_1.default is not a function
```

実行時エラーが出たはずです。

コード生成時に相互運用性を担保する仕組みが働いていないことがわかります。

## 最後に

ここまで読んでくれてありがとうございます！

自分でも文章がグチャグチャで分かりにくいだろうなというのを痛感しています。

Twitterもしているので、よかったら仲良くしてください！

[@omochizou](https://twitter.com/omochizou)

