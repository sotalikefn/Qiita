---
title: 【画面比率ですべて伸縮】レスポンシブを@mixinと@functionで簡単に実現する
tags:
  - CSS
  - scss
  - レスポンシブ
  - mixin
  - Function
private: false
updated_at: '2024-07-17T13:29:44+09:00'
id: a379e5951a8d7eb4f9a5
organization_url_name: null
slide: false
ignorePublish: false
---
レスポンシブデザインにおいて、デザインされたアートボードサイズ以外のレイアウトをどうすればよいか、デザイナーとコーダーのよくある問題だと思います。

デザインするときにそこまで考慮していたら単調なデザインになるし、苦労してコーディング終わらせてから想定外なフィードバックが返ってきたり...




# レスポンシブのための便利な`@mixin`、`@function`を考える

開発者目線ですが、体感的にデザイナーはデザインしたサイズ感でwindow.widthに合わせて伸縮するのを好む傾向にあると思います。
ですので、デザインのサイズ感を極力遵守する形でレスポンシブを実現できる便利な`@mixin`、`@function`を考えてみたいと思います。




# 実現したいこと
![レスポンシブ](https://i.gyazo.com/2c0222d4616a75f638d59ed6f66c3528.jpg)

- window.widthに対して各要素のアスペクト比を保って伸縮したい。
- 一定のwindow.width以上は伸縮をさせないように上限を設ける。
- 伸縮させるとどうしてもデザインが破綻する場合があるので、最小値・最大値を考慮する。
- stylelintのCSSプロパティソートの邪魔をしない。

## こちらは別の機会に...
- font-sizeもいい感じに伸縮したい。
  - ブラウザは10px未満のfont-sizeをデフォルトでは描画しない（強制的に10pxになる）ので、font-sizeの減縮には限度があることを考慮する。



# デザインデータ
通常Tabletのデザインまで用意することは少ないですが、デザインによってはTabletでブレークポイントを設けたいケースもあると思います。
そこで今回はSP、Tablet、Desktopの3パターンのブレークポイントを設ける形で考えます。

|デバイス|アートボードサイズ|
|---|---|
|SP|375px|
|Tablet|768px|
|Desktop|1600px|

※ 1600px以上はコンテンツブロックをセンタリングする想定





# 全容
```scss
$device: 0px;
$mobile: 375px;
$tablet: 768px;
$desktopS: 1024px;
$desktop: 1600px;

@mixin onlyTablet {
  @media only screen and (min-width: $tablet) and (max-width: $desktopS) {
    $device: $tablet !global;
    @content;
    $device: 0px !global;
  }
}

@mixin onlyDesktop {
  @media only screen and (min-width: $desktopS + 1px) {
    $device: $desktopS !global;
    @content;
    $device: 0px !global;
  }
}

@function vw($px) {
  @if $device == $desktopS {
    @return clamp(calc($desktopS / $desktop * $px), calc($px / $desktop * 100vw), $px);
  }
  @else if $device == $tablet {
    @return clamp($px, calc($px / $tablet * 100vw), calc($desktopS / $tablet * $px));
  }

  @return clamp($px, calc($px / $mobile * 100vw), calc($tablet / $mobile * $px));
}


.hogehoge {
  padding-top: vw(20px);

  @include onlyTablet {
    padding-top: vw(60px);
  }

  @include onlyDesktop {
    padding-top: vw(120px);
  }
}
```


# コンパイル結果
```css
.hogehoge {
  padding-top: clamp(20px, 5.3333333333vw, 40.96px);
}
@media only screen and (min-width: 768px) and (max-width: 1024px) {
  .hogehoge {
    padding-top: clamp(60px, 7.8125vw, 80px);
  }
}
@media only screen and (min-width: 1025px) {
  .hogehoge {
    padding-top: clamp(76.8px, 7.5vw, 120px);
  }
}
```


# 定数
各ブレークポイントを定数化しておくことで、後々の変更に耐えられる作りにします。
`$device`には後述の[!globalフラグ](#globalフラグ)によってdevice状態が入ってきます。


```scss
$device: 0px; // デバイスの振り分け状態を格納するグローバル変数
$mobile: 375px; // SPのデザインデータサイズ
$tablet: 768px; // Tablet下限ブレークポイント
$desktopS: 1024px; // Tablet上限、Desktopの下限ブレークポイント
$desktop: 1600px; // Desktop上限ブレークポイント
```





# mixin
メディアクエリmixinを定義。
ポイントは[!globalフラグ](#globalフラグ)でグローバル変数として定義してある`$device`に状態を一時保存する部分です。
また、Content Blocksでの処理が終わったら改めて`$device`を初期値に戻してあげる必要があります。

```scss
@mixin onlyTablet {
  // @media only screen and (min-width: 768px) and (max-width: 1024px) {
  @media only screen and (min-width: $tablet) and (max-width: $desktopS) {
    // $device: 768px !global;
    $device: $tablet !global;
    @content;
    $device: 0px !global;
  }
}

@mixin onlyDesktop {
  // @media only screen and (min-width: 1024px + 1px) {
  @media only screen and (min-width: $desktopS + 1px) {
    // $device: 1024px !global;
    $device: $desktopS !global;
    @content;
    $device: 0px !global;
  }
}
```


## !globalフラグ
> If you need to set a global variable’s value from within a local scope (such as in a mixin), you can use the !global flag. A variable declaration flagged as !global will always assign to the global scope.

> ローカルスコープからグローバル変数の値を設定する必要がある場合（ミックスインなど）、!globalフラグを使うことができる。!globalフラグが付いた変数宣言は、常にグローバルスコープに代入されます。

[SASSドキュメント - Variables - ScopeScope](https://sass-lang.com/documentation/variables/#scope)


# function
`$device`で渡された引数によって、SP、Tablet、Desktopのそれぞれの最小値・推奨値・最大値を算出し、[clamp関数](https://developer.mozilla.org/ja/docs/Web/CSS/clamp)を返却します。


```scss
@function vw($px) {
  @if $device == $desktopS {
    // clamp(calc(1024px / 1600px * 120px), calc(120px / 1600px * 100vw), 120px);
    // => clamp(76.8px, 7.5vw, 120px);
    @return clamp(calc($desktopS / $desktop * $px), calc($px / $desktop * 100vw), $px);
  }
  @else if $device == $tablet {
    // clamp(60px, calc(60px / 768px * 100vw), calc(1024px / 768px * 60px));
    // => clamp(60px, 7.8125vw, 80px);
    @return clamp($px, calc($px / $tablet * 100vw), calc($desktopS / $tablet * $px));
  }

  // clamp(20px, calc(20px / 375px * 100vw), calc(768px / 375px * 20px));
  // => clamp(20px, 5.3333333333vw, 40.96px);
  @return clamp($px, calc($px / $mobile * 100vw), calc($tablet / $mobile * $px));
}
```

## SPの場合
```scss
// clamp(20px, calc(20px / 375px * 100vw), calc(768px / 375px * 20px));
// => clamp(20px, 5.3333333333vw, 40.96px);
@return clamp($px, calc($px / $mobile * 100vw), calc($tablet / $mobile * $px));
```

**最小値**
　指定した20px。

**推奨値**
　20pxと基準サイズの375pxの比率を算出し、100vwを掛けることで画面幅に対する数値をvwで得ることができる。

**最大値**
  Tablet最小値の768pxまで伸縮するため、`$tablet / $mobile`で拡大率を算出し、20pxに掛けることで768pxまで大きくした場合の数値を得ることができる。


## Tabletの場合
```scss
// clamp(60px, calc(60px / 768px * 100vw), calc(1024px / 768px * 60px));
// => clamp(60px, 7.8125vw, 80px);
@return clamp($px, calc($px / $tablet * 100vw), calc($desktopS / $tablet * $px));
```

**最小値**
　指定した60px。

**推奨値**
　60pxと基準サイズの768pxの比率を算出し、100vwを掛けることで画面幅に対する数値をvwで得ることができる。

**最大値**
  Desktop最小値の1024pxまで伸縮するため、`$desktopS / $tablet`で拡大率を算出し、60pxに掛けることで1024pxまで大きくした場合の数値を得ることができる。



## Desktopの場合
```scss
// clamp(calc(1024px / 1600px * 120px), calc(120px / 1600px * 100vw), 120px);
// => clamp(76.8px, 7.5vw, 120px);
@return clamp(calc($desktopS / $desktop * $px), calc($px / $desktop * 100vw), $px);
```

**最小値**
  Desktop最小値の1024pxまで伸縮するため、`$desktopS / $desktop`で縮小率を算出し、120pxに掛けることで1024pxまで小さくした場合の数値を得ることができる。

**推奨値**
　120pxと基準サイズの1600pxの比率を算出し、100vwを掛けることで画面幅に対する数値をvwで得ることができる。

**最大値**
　指定した120px。


# CSS
`padding-top`などプロパティ名は通常通り書くことでstylelintの邪魔をせずにプロパティの並び順などを統一化できます。
また、function名はシンプルに`vw()`とし、SP、Tablet、Desktopで統一したfunctionの呼び方ができるようにすることで、VS Codeのスニペットにも登録しやすくしています。

```scss
.hogehoge {
  padding-top: vw(20px);

  @include onlyTablet {
    padding-top: vw(60px);
  }

  @include onlyDesktop {
    padding-top: vw(120px);
  }
}
```

:::note alert
こんな感じに書くとプロパティ名でソートされず（`@include`でソートされてしまう）、プロパティ名の並び順を自動で統一できない。
```scss
.hogehoge {
  @include vw(padding-top, 20px, 60px, 120px);
}
```
:::


# 参考
[SASS ドキュメント](https://sass-lang.com/documentation/)
[現場で役立つ実践Sass（3）変数を使いこなす](https://blog.adobe.com/jp/publish/2016/03/14/web-practical-sass-03-using-variables)
