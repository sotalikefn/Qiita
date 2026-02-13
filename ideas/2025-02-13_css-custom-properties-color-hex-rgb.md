# 記事アイデア：CSSカスタムプロパティで色を定義するときの便利な書き方

## 基本情報

- 作成日: 2025-02-13
- ステータス: [ ] アイデア段階 [ ] 調査中 [ ] 執筆中 [x] 完了

## 概要

CSSのカスタムプロパティで色を定義する際、SCSSの `@each` と色関数（`red()`, `green()`, `blue()`）を使って、HEX と RGB の両方を一括で出力する書き方を紹介する。各色を HEX と RGB の両方で手書きする手間を省き、`color: var(--white)` と `background: rgb(var(--white-rgb) / 0.4)` の両方で使えるようにする。

## 問題提起（なぜこの書き方か）

**RGB だけ定義する場合**

```css
:root {
  --white: 255 255 255;
}
```

このように書くと、不透明な色でも `color: rgb(var(--white))` と必ず `rgb()` で囲む必要があり、記述が冗長になる。

**HEX だけ定義する場合**

```css
:root {
  --white: #fff;
}
```

このように書くと、`color: var(--white)` とは書けるが、`background: rgb(var(--white) / 0.4);` のように透明度を付けたいときに書けない（HEX は `rgb(〜 / 0.4)` の形で使えない）。

**そこで**、`--white`（HEX）と `--white-rgb`（R G B のスペース区切り）の両方を用意するこのアイデアの書き方にすると、不透明なときは `color: var(--white)`、透明度が必要なときは `background: rgb(var(--white-rgb) / 0.4);` と、用途に合わせて使い分けられる。

## 想定読者

- SCSS/CSS でデザインシステムやテーマを組んでいるフロントエンド開発者
- カスタムプロパティで色を管理したい開発者
- 透明度付きの色（`rgb(var(--x-rgb) / 0.4)`）を使いたい開発者

## 主なポイント

1. 色マップ（`$colors`）を1箇所で定義し、HEX と RGB の両方の変数を自動生成する
2. `:root` で `--{name}`（HEX）と `--{name}-rgb`（R G B スペース区切り）を出力する書き方
3. 呼び出し側では `color: var(--white)` と `background: rgb(var(--white-rgb) / 0.4)` の両方が使える
4. 各色を HEX と RGB で二重に書く必要がなくなり、保守性が上がる

## サンプルコード

```scss
$colors: (
  white: #fff,
  black: #3b3b3b,
  red: #dc3232,
  blue: #476ba6,
);

:root {
  @each $name, $color in $colors {
    --#{$name}: #{$color};
    --#{$name}-rgb: #{red($color)} #{green($color)} #{blue($color)};
  }
}
```

利用例：

- `color: var(--white);`
- `background: rgb(var(--white-rgb) / 0.4);`（透明度付き）

## 参考資料

- （執筆時にSCSS色関数・カスタムプロパティのMDN等を追加）

## メモ

- 各色を HEX と RGB の両方で書く必要がなく、一括で管理できる
- `--{name}-rgb` は「R G B」のスペース区切りなので、`rgb(var(--white-rgb) / 0.4)` のようにそのまま使える

## TODO

- [x] 執筆
- [x] 参考資料のリンク追加
