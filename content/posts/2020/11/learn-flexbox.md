---
title: "CSSで要素２つを縦並びにしつつ中央寄せする方法とFlexbox微入門😾"
date: 2020-11-24T16:56:03+09:00
draft: false
tags: ["CSS"]
---

HTMLのブロック要素２つを縦並びにして中央に配置するというスタイリングをCSSでする必要があったため、それを実現するまでに調べた事を自分用にまとめます。

"縦並びにして中央に配置(中央寄せ)する"とは下図を指します。(大雑把です)

![example](https://user-images.githubusercontent.com/37544784/100083339-abf21400-2e8c-11eb-8466-a8e68bd42feb.png)

## ようするにどうすればいいか

今回はFlexboxを使用しました。

```html
<div class="container">
    <div>one</div>
    <div>two</div>
</div>
```

```css
.container {
    display: flex;
    flex-direction: column;
    align-items: center;
}
```

これで上の画像のようなレイアウトは実現できました

ここからは良い機会なので、少しFlexboxについて学んだことをまとめたものと、なぜこのCSSでこのようなレイアウトが実現できるか[MDN](https://developer.mozilla.org/ja/docs/Learn/CSS/CSS_layout/Flexbox)を眺めながら学んだことを書いていきます。

## What is Flexbox anyway?

Flexboxはあくまで省略表現にすぎず、MDN公式では**CSS Flexible box layout**と書かれています。

ですがこれでは長いので、以後Flexboxと表記していきます。



Flexibleとは日本語で、「**曲げやすい しなやかな 柔軟な**」という意味を持ちます。これだけでFlexboxがどういう事を実現したいかが少し見えてきそうです。

Flexboxとは一次元のアイテムのレイアウトを柔軟に(flexible)定義できるCSSのモジュールの一つです。[^1]

[^1]: https://developer.mozilla.org/ja/docs/Web/CSS/CSS_Flexible_Box_Layout

Flexboxを使用しレイアウトされる領域は**フレックスコンテナー**[^2]と呼ばれ、フレックスコンテナーの子要素はフレックスアイテムと呼ばれるモノに変化し、フレックスアイテムは任意の方向に柔軟に動かせるようになります。

[^2]: https://developer.mozilla.org/ja/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox#The_flex_container

また、余白を埋めるため伸ばしたり、親からあふれる部分を収縮したりflexibleに操る事も出来ます

## 例と共に学ぶ

MDNにちょうどいい例があったので、これをもう少し深堀りして学んでいきます。

例はこちらです。

```html
<div class="box">
	<div>One</div>
    	<div>Two</div>
    	<div>Three
              <br>has
              <br>extra
              <br>text
		</div>
	</div>
</div>
```

```css
.box {
    display: flex;
    justify-content: space-between;
}
      
```

![mdn-example](https://user-images.githubusercontent.com/37544784/100133190-c7c7db00-2ec9-11eb-953d-ac842e7dcb85.png)

このコードでなぜこのレイアウトが実現できるか少し調べていきましょう。



まずFlexboxを使用する第一歩として、フレックスコンテナーと呼ばれるFlexboxを使用する領域を作成する必要があります。

そのためコンテナとなる要素を作り、その要素にdisplay: flex;を使用する必要があります。[^3]

[^3]: https://developer.mozilla.org/ja/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox#The_flex_container

```html
<div class="box">
 ....
</div>
```

```css
.box {
	display: flex;
}
```

上の例ではclassとしてboxを宣言し、そこにflexを指定しています。

すると下図の状態になります。

![mdn-example2](https://user-images.githubusercontent.com/37544784/100133381-09588600-2eca-11eb-9845-927241e760ca.png)

フレックスコンテナを作成するとコンテナ直下の子要素がフレックスアイテムと呼ばれるものに変化します。この時フレックスアイテムは決まった振る舞いをします。この挙動を元になぜ上のようなレイアウトになったか見ていきましょう。



まずフレックスコンテナを作成すると、コンテナに含まれるフレックスアイテムはおおまかに以下の様に振る舞います。

- フレックスアイテムは**flex-direction**プロパティ(主軸)の定義に沿って並んで表示される
- フレックスアイテムは**主軸**の先頭に並びます
- フレックスアイテムは主軸方向に伸縮はしないが収縮したりする
- フレックスアイテムは**交差軸**の大きさを埋めるように伸縮する(align-itemsプロパティの定義)

上のレイアウトを説明するのに必要そうな挙動を挙げていきました。他にもまだありますが割愛します。

これらはMDN公式ドキュメントで説明されています。(https://developer.mozilla.org/ja/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox#The_flex_container)



まず最初から見ていきましょう。

**フレックスアイテムは**flex-direction**プロパティ(主軸)の定義に沿って並んで表示される**

フレックスコンテナを作成しフレックスアイテムとなるとまずフレックスアイテムはflex-directionプロパティの定義に沿って並んで表示されます。 ではflex-directionプロパティとは何でしょうか。 これはFlexboxにおける**主軸**方向を定義しそれを元にフレックスコンテナ内のフレックスアイテムを配置する方法を決めるプロパティです。[^4]

[^4]: https://developer.mozilla.org/ja/docs/Web/CSS/flex-direction

主軸(Main Axis)とはflex-directionプロパティによって設定された値です。

この主軸がとりうる値は以下の通りです[^5]

- row
- row-reverse
- column
- column-reverse

[^5]: https://developer.mozilla.org/ja/docs/Glossary/Main_Axis

主軸(flex-direction)の既定値(初期値)は**row**です。[^6]

[^6]: https://developer.mozilla.org/ja/docs/Web/CSS/flex-direction#Formal_definition

つまり上のレイアウトは、最初にflex-directionの値によって配置される方法が決まっているので初期値rowに従い横向きに並ぶという事が理解できます。

そして次の

**フレックスアイテムは**主軸**の先頭に並ぶ**

という箇所でrowの先頭(左端)に圧縮され並びます。

**フレックスアイテムは主軸方向に伸縮はしないが収縮したりする**

は今回あまり関係ないので飛ばします。

そして最後の

**フレックスアイテムは**交差軸**の大きさを埋めるように伸縮する**

でフレックスコンテナの高さ分埋めるように伸縮されていることがわかります。

これはalign-itemsというプロパティの初期値が**normal**に設定されており、フレックスコンテナ下では

normalは**stretch**という値と同じ振る舞いをするためです。stretchは幅や高さの制約の範囲内でラインと同じになるように伸縮するという振る舞いをします。

そして、Flexboxにおける**交差軸**(Cross Axis)とは主軸(今回はrow)の反対を指します。[^7]

[^7]: https://developer.mozilla.org/ja/docs/Glossary/Cross_Axis

交差軸方向(column)の高さいっぱいにラインと同じになるまで伸縮しているという事です。

これでひとまず上のレイアウトの説明はできました。

あとはMDNの例に近づくようにjustify-contentプロパティを追加しましょう

```
.box {
	display: box;
	justify-content: space-between;
}
```

すると例の通り

![mdn-example](https://user-images.githubusercontent.com/37544784/100133190-c7c7db00-2ec9-11eb-953d-ac842e7dcb85.png)

となります。

なぜjustify-content: space-between;を追加するだけでこうなるのか見ていきましょう。



### justify-contentの効果

justify-contentプロパティとは主軸に沿ってフレックスアイテムの配置する方法を決めるものです。

今回、flex-directionは弄っていないため、初期値であるrowが適用され主軸もrowとなります。

今回はjustify-contentの値はspace-betweenです。これはフレックスアイテムがフレックスコンテナの中における主軸方向に均等に配置されるものです。最初のアイテムは主軸先頭に、最後のアイテムは主軸最後に置かれます。そして、間のアイテム同士の間隔は等間隔に置かれます。

確かに例の画像と挙動が一致しています。

##最後に

今まで学んだ事を元に、もともとやりたかったレイアウトである

![example](https://user-images.githubusercontent.com/37544784/100083339-abf21400-2e8c-11eb-8466-a8e68bd42feb.png)

が

```html
<div class="container">
    <div>one</div>
    <div>two</div>
</div>
```

```css
.container {
	display: flex;
    flex-direction: column;
    align-items: center;
}
```

でなぜ実現できるか解いていこうと思います。

まず、このレイアウトにするには

1. ２つの要素をフレックスにして、主軸が初期値でrowなので横並びになってるため、主軸をcolumnにして縦並びにする。
2. それを主軸方向のスタイリングをするプロパティでcenteredする

で実現できそうです。



1を実現するには

```css
display: flex;
```

は前提として、並びが主軸の初期値rowに沿って横並びになっているため主軸をcolumnにして縦並びにする必要があります。ですから

```css
flex-direction: column;
```

で主軸をcolumnに変えるためflex-directionプロパティを使っていることが理解できます。

あとは2を実現するため、先程学んだ主軸方向のスタイリングを変えられる**justify-content**を使って

```css
justify-content: center;
```

とします。

これで無事

```css
.container {
	display: flex;
    flex-direction: column;
    align-items: center;
}
```

が

![example](https://user-images.githubusercontent.com/37544784/100083339-abf21400-2e8c-11eb-8466-a8e68bd42feb.png)

になる理由がわかりました。

今回アウトラインを書いてから記事を書いていないためグチャグチャでとても分かりづらいと思います。それでも最後まで読んでくださってありがとうございます。

