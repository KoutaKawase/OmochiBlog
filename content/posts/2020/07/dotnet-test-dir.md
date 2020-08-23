---
title: ".NETCore上のC#でテストコードからテストプロジェクトのディレクトリを取得する方法"
date: 2020-07-07T06:36:20+09:00
draft: false
tags: ["Ubuntu", "NETCore", "Csharp", "NUnit"]
---

## 環境

Linux Ubuntu 18.04

.NET Core 3.1

NUnit 3.12.0

## 結論

```C#
AppContext.BaseDirectory.Substring(0, AppContext.BaseDirectory.IndexOf("bin"));
```



でテストプロジェクトの絶対パスが手に入る。



例えば

```bash
~/work/hogeProgram/tests/hogeTests.cs
```

みたいな構造になっているとすると、hogeTests.cs内で

```C#
AppContext.BaseDirectory
```

を利用しても

```
~/work/hogeProgram/tests/bin/Debug/netcoreapp3.1
```

みたいにアセンブリーが入ったディレクトリを取得してきてしまう。



従って bin以降のパスを削除すればテストプロジェクトの絶対パスが手に入ります。
