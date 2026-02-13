---
title: CSSカスタムプロパティで色を定義するとき、HEXとRGBを一括で出力する書き方
tags:
  - CSS
  - scss
  - フロントエンド
  - デザインシステム
  - カスタムプロパティ
private: false
updated_at: '2026-02-13T12:03:12+09:00'
id: 123fc27d18ed4ad70262
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

デザインシステムやテーマでCSSのカスタムプロパティ（CSS変数）を使って色を管理するとき、**不透明な色**（`color: var(--white)`）と**透明度付きの色**（`background: rgb(var(--white-rgb) / 0.4)`）の両方を使い分けたい場面は多いと思います。

本記事では、**色を1箇所で定義するだけで**、HEXとRGBの両方のカスタムプロパティを自動生成する書き方を紹介します。SCSSの`@each`と色関数（`red()`, `green()`, `blue()`）を使い、各色をHEXとRGBで二重に書く手間を省きます。

## 想定読者

- SCSS/CSSでデザインシステムやテーマを組んでいるフロントエンド開発者
- カスタムプロパティで色を一元管理したい開発者
- `rgb(var(--x-rgb) / 0.4)` のような透明度付きの色を使いたい開発者

## 前提条件

- SCSSが使える環境
- カスタムプロパティの基本的な使い方を理解していること

## HEXだけ・RGBだけだとどう困るか

### RGBだけ定義する場合

```css
:root {
  --white: 255 255 255;
}
```

この場合、不透明な色でも `color: rgb(var(--white))` と必ず `rgb()` で囲む必要があり、記述が冗長になります。

### HEXだけ定義する場合

```css
:root {
  --white: #fff;
}
```

この場合、`color: var(--white)` とは書けますが、`background: rgb(var(--white) / 0.4);` のように透明度を付けたいときには書けません。`rgb(R G B / alpha)` のように透明度をスラッシュで指定する記法では、**変数の中身はRGBのスペース区切り**である必要があり、HEX値をそのまま渡すことはできないためです。

### 両方あるとどうなるか

`--white`（HEX）と `--white-rgb`（RGB）の**両方**を用意すると、

- 不透明なとき → `color: var(--white)`
- 透明度が必要なとき → `background: rgb(var(--white-rgb) / 0.4);`

と、用途に応じて使い分けられます。

## 実装：色マップからHEXとRGBを一括出力

色マップ（`$colors`）を1箇所で定義し、`@each` でループしながら HEX と RGB の両方を出力します。

```scss
$colors: (
  'white': #fff,
  'black': #3b3b3b,
  'red': #dc3232,
  'blue': #476ba6,
);

:root {
  @each $name, $color in $colors {
    --#{$name}: #{$color};
    --#{$name}-rgb: #{red($color)} #{green($color)} #{blue($color)};
  }
}
```

コンパイル後のCSSは次のとおりです。

```css
:root {
  --white: #fff;
  --white-rgb: 255 255 255;
  --black: #3b3b3b;
  --black-rgb: 59 59 59;
  --red: #dc3232;
  --red-rgb: 220 50 50;
  --blue: #476ba6;
  --blue-rgb: 71 107 166;
}
```

`--{name}-rgb` はRGBのスペース区切りなので、`rgb(var(--white-rgb) / 0.4)` のようにそのまま使えます。

### 利用例

```css
/* 不透明な色 */
color: var(--white);
background: var(--blue);

/* 透明度付きの色 */
background: rgb(var(--white-rgb) / 0.4);
background: rgb(var(--red-rgb) / 0.2);
border: 2px solid rgb(var(--black-rgb) / 0.1);
```

## この書き方のメリット

1. **1箇所で管理**：色マップを修正するだけで、HEXとRGBの両方が更新される
2. **保守性の向上**：各色をHEXとRGBで二重に書く必要がなくなる
3. **使い分けが明確**：不透明な用途は `var(--name)`、透明度が必要な用途は `rgb(var(--name-rgb) / 0.4)` と用途に応じて選べる

## まとめ

SCSS の `@each` と `red()` / `green()` / `blue()` を使うことで、**色定義1つからHEXとRGBの両方のカスタムプロパティを自動生成**できます。デザインシステムやテーマの色管理を、少ない記述でシンプルに保てます。

## 参考資料

- [Using CSS custom properties (variables) - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/Using_CSS_custom_properties)
- [var() - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/var)
- [Sass: Built-In Modules - sass:color](https://sass-lang.com/documentation/modules/color)
- [rgb() - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/color_value/rgb)
