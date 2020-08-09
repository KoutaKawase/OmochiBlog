---
title: "TypeScriptのライブラリの型定義を拡張する方法(with Discord.js)"
date: 2020-08-09T18:22:40+09:00
draft: false
tags: ["TypeScript"]
---

最近Discord.jsでDiscordのBot(身内用)をTypeScriptで作っています。その中でDiscord.jsの型定義に追加したい型が出てきました。

```typescript
import Discord from 'discord.js';
import { Command } from './commands/types/Command';

const client = new Discord.Client();
client.commands = new Discord.Collection<string, Command>();
```

こんな感じで後からDiscord.Clientオブジェクトにcommandsというプロパティを勝手に追加してるんですが、もちろんこんなものはdiscord.jsのDiscord.Clientの型定義に含まれていないので、自分で拡張してやる必要があります。

今回はその型を自分で拡張する方法を自分用にまとめておきます。

## 結論

独自の型定義ファイルを格納するためのディレクトリをsrcディレクトリ内に作成します。

```bash
mkdir src/types
```

独自の型定義ファイルを作成するため.d.tsファイルを作成。

```bash
touch src/types/client.d.ts
```

ファイルを開き、ファイル内で

```typescript
import { Client, Collection } from 'discord.js';
import { Command } from './commands/types/Command'; //これはあんま今関係ない

declare module 'discord.js' {
    interface Client {
        commands: Collection<string, Command>;
    }
}
```

を記述。

あとは保存すれば自動で反映され

![Screenshot from 2020-08-09 19-10-18](https://user-images.githubusercontent.com/37544784/89729804-1913a200-da74-11ea-96e5-5fe582247861.png)

型がついてることがわかります。

## 解説

TypeScriptでは実装を定義しない宣言(型のみの宣言)*を*アンビエント宣言**と呼びます。

アンビエント宣言は .d.tsという拡張子を持ったファイルで宣言できます。

C++における .hファイルのようなものです。

今回はこのアンビエント宣言を利用し型を定義しました。

### declare moduleとは

```typescript
declare module "名前"
```
をアンビエント宣言内で記述することで "名前" で指定した型定義モジュールを指定することができます。

今回はdiscord.jsの型を弄りたかったので、declare module名にdiscord.jsを指定し、Clientの型を拡張しています。

### interfaceのマージ

TypeScriptではinterfaceのマージが可能です。

例えば

```typescript
interface Foo {
    bar: string;
    baz: string;
}
```

というinterfaceが存在した場合

```typescript
//前宣言したやつ
interface Foo {
    bar: string;
    baz: string;
}

//新しい宣言
interface Foo {
    hoge: string;
}
```

と新しくプロパティを追加してinterfaceを宣言すると

```typescript
interface Foo {
    bar: string;
    baz: string;
    hoge: string;
}
```



となり、マージされた形にすることができます。

今回はこれを利用し

```typescript
interface Client {
    commands: Collection<string, Command>;
}
```

としています。discord.jsにすでに定義されているClientの型にマージしています。

これらのようなマージをTypeScriptでは [Declaration Merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html)と言います。

## お礼
最後まで読んでくださりありがとうございます。
わかりにくい文章かもしれませんが、お役に立てたら嬉しいです。
