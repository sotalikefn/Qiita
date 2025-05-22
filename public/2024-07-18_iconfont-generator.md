---
title: Webサービスを使わずにローカルでアイコンフォントをSVGから生成しよう【gulp-iconfont × gulp-consolidateの代替】
tags:
  - webfont
  - gulp
  - iconfont
  - アイコンフォント
  - fantasticon
private: false
updated_at: '2024-07-18T13:11:50+09:00'
id: 91dfd2bf40f2fef63225
organization_url_name: null
slide: false
ignorePublish: false
---
アイコンフォントを使用する際、[Font Awesome](https://fontawesome.com/)や[icomoon](https://icomoon.io/)、[fontello](https://fontello.com/)などのWebサービスを使用している方も多いかと思いますが、私は、毎回プロジェクトごとにWebサービスにアップロードしてフォントファイルをダウンロードしてという作業が面倒で、Webサービスを使わずにローカルでフォントファイルをSVGから生成しています。

# いにしえの技術
一昔前は[gulp-iconfont](https://www.npmjs.com/package/gulp-iconfont)と[gulp-consolidate](https://www.npmjs.com/package/gulp-consolidate)を使用してSVGからWebフォントを生成する方法がよく見られました。

![検索結果](https://i.gyazo.com/3da7d5952e82d43da428bf696c92ea21.png)

gulpでスクリプトを書けるので、細かいところ（フォントファイルやSCSSの出力先など）まで自由にコントロールすることができました。
しかし、これらのパッケージはメンテナンスされておらず、**gulp v5.0.0で動きません。**
（v4.0.2までは動きますが...）

- [gulp-iconfont](https://www.npmjs.com/package/gulp-iconfont)（Last publish 3 years ago）
- [gulp-consolidate](https://www.npmjs.com/package/gulp-consolidate)（**Last publish 8 years ago**）


これらを何に代替したら良いかという記事をあまり見かけないため、まとめてみたいと思います。

# 実現したいこと
- メンテナンスされているパッケージを使用する。
- [gulp-iconfont](https://www.npmjs.com/package/gulp-iconfont)と[gulp-consolidate](https://www.npmjs.com/package/gulp-consolidate)で生成していた時の使用感を極力保つ。
  - SVGからアイコンフォントを生成する。
  - アイコン名はSVGのファイル名を流用する。
  - CSS（SCSS）のテンプレートを作ることができる。
- CLI（package > scripts）でビルドすることができる。





# 代替パッケージの選定
SVGからアイコンフォントを生成するパッケージはいくつか存在します。

## webfont
https://www.npmjs.com/package/webfont

:::note alert
- READMEの内容と実際の挙動が異なる部分がある
- CLIで指定できる内容が限られている
- バグ報告が放置されておりメンテナンスされていなそう
:::

## svgtofont
https://www.npmjs.com/package/svgtofont

:::note alert
- CLIが無さそう？
- Webサービスと連携してアイコンフォントを生成するような仕様？
:::

## fantasticon
https://www.npmjs.com/package/fantasticon

:::note info
- CLIをメインとしている
- run commandで設定を書くことができる
- CSS（SCSS）テンプレートも作ることができる（Handlebars）
:::


ということで[fantasticon](https://www.npmjs.com/package/fantasticon)でアイコンフォントを生成する方法をご紹介します。



# 準備
サンプルプロジェクトのファイル構成は下記になります。

```
.
├─ dist                        // Document Root
|  └─ assets
|     └─ font                  // ファイル出力先
├─ package.json
└─ src
   └─ font
      └─ svg                   // アイコンフォントにしたいSVGを格納
         ├─ blank.svg
         ├─ pdf.svg
         └─ ...
```

# パッケージのインストール
```
$ npm i -D fantasticon
```

# CLIでアイコンフォントを生成してみる
[fantasticon](https://www.npmjs.com/package/fantasticon)はCLIで完結するので、まずは試しにコマンドでアイコンフォントを生成してみましょう。

```
$ npx fantasticon src/font/svg/ -o dist/assets/font/

Generating font kit...
✔ 6 SVGs found in src/font/svg/
✔ Generated dist/assets/font/icons.eot
✔ Generated dist/assets/font/icons.woff2
✔ Generated dist/assets/font/icons.woff
✔ Generated dist/assets/font/icons.css
✔ Generated dist/assets/font/icons.html
✔ Generated dist/assets/font/icons.json
✔ Generated dist/assets/font/icons.ts
Done

$ 
```

するとアイコンフォントが生成されます。

```
.
├─ dist
|  └─ assets
|     └─ font
|        ├─ icons.css    // 生成されたファイル
|        ├─ icons.eot    // 生成されたファイル
|        ├─ icons.html   // 生成されたファイル
|        ├─ icons.json   // 生成されたファイル
|        ├─ icons.ts     // 生成されたファイル
|        ├─ icons.woff   // 生成されたファイル
|        └─ icons.woff2  // 生成されたファイル
├─ package.json
└─ src
   └─ font
      └─ svg
         ├─ blank.svg
         ├─ pdf.svg
         └─ ...
```

毎回このコマンドを実行するのは億劫なので、`package > scripts`に登録しておきましょう。

```package.json
{
  ...
  "scripts": {
    "iconfont": "npx fantasticon src/font/svg/ -o dist/assets/font/"
  }
}
```

これで次回以降はもっとシンプルなコマンドでアイコンフォントが生成されます。

```
$ npm run iconfont
```

生成された`dist/assets/font/icons.html`をブラウザにドラッグ&ドロップしてみてください。
生成したアイコンを一覧で見ることができると思います。

![プレビュー](https://i.gyazo.com/ca2ec64a1329068da9a360a8da3e7375.png)


# 設定をカスタマイズ
`fantasticon`コマンドに渡したオプションは、run commandファイルで設定ファイル化しておくことができます。

```.fantasticonrc
{
  "name": "iconfont",               // アイコンフォントファイル名・font-famiry名
  "inputDir": "src/font/svg",       // SVG格納ディレクトリパス
  "outputDir": "dist/assets/font",  // 出力先ディレクトリパス
  "fontsUrl": "/assets/font",       // アイコンフォントの設置パス（絶対パスでも相対パスでも可）
  "normalize": true,                // アイコンを最も大きいアイコンの高さにスケーリングして正規化する
  "fontTypes": [                    // 生成するフォントタイプ
    "eot",
    "woff",
    "woff2"
  ],
  "assetTypes": [                   // 生成するファイル
    "scss",
    "css",
    "html"
  ],
  "templates": {                    // 生成するファイルのテンプレート
    "scss": "src/font/templates/iconfont.scss.hbs",
    "html": "src/font/templates/iconfont.html.hbs"
  }
}
```

これをプロジェクトルートに設置しておけば、`package > scripts`の内容はもっとシンプルになります。

```package.json
{
  ...
  "scripts": {
    "iconfont": "npx fantasticon"
  }
}
```

この設定で下記のようにフォントファイルが生成されます。

```
.
├─ .fantasticonrc              // run command設定ファイル
├─ dist                        // Document Root
|  └─ assets
|     └─ font
|        ├─ iconfont.css       // 生成されたファイル
|        ├─ iconfont.eot       // 生成されたファイル
|        ├─ iconfont.html      // 生成されたファイル
|        ├─ iconfont.scss      // 生成されたSCSS
|        ├─ iconfont.woff      // 生成されたファイル
|        └─ iconfont.woff2     // 生成されたファイル
├─ package.json
└─ src
   ├─ css
   └─ font
      ├─ svg                   // アイコンフォントにしたいSVGを格納
      │  ├─ blank.svg
      │  ├─ pdf.svg
      │  └─ ...
      └─ templates
         ├─ iconfont.html.hbs  // プレビュー用HTMLテンプレート
         └─ iconfont.scss.hbs  // SCSSテンプレート
```

:::note warn
fontsUrlオプションでアイコンフォントのパスを絶対パスにしたので、プレビューHTMLをブラウザにドラッグ&ドロップしても見れなくなります。
ローカルホストが立ち上がっていれば、`http://localhost/assets/font/iconfont.html`で見ることができます。
:::

# テンプレート
設定ファイルのtemplatesプロパティでテンプレートを指定することで、生成されるSCSSの内容を自由にカスタマイズすることができます。
このテンプレートはHandlebarsで記述します。

デフォルトのテンプレートはGitHubリポジトリから確認することができます。

https://github.com/tancredi/fantasticon/tree/master/templates


# 生成されたSCSSをsrcディレクトリに移動したい
生成されたファイルの中に`iconfont.scss`があります。
SCSSはsrcディレクトリで他のSCSSと一緒にビルドしますので、distディレクトリに存在するのはふさわしくありません。

[move-file-cli](https://www.npmjs.com/package/move-file-cli)を使って移動します。

https://www.npmjs.com/package/move-file-cli

:::note warn
[fantasticon](https://www.npmjs.com/package/fantasticon)の惜しいところとして、ファイルごとに出力先を指定できるようなオプションがあったらいいのになぁ...と思いました（あるのかな？）。
:::

```
$ npm i -D move-file-cli
```

```package.json
{
  ...
  "scripts": {
    "iconfont": "npx fantasticon && npx move-file dist/assets/font/iconfont.scss src/css/foundation/iconfont.scss"
  }
}
```

これでiconfont.scssを所定の位置に移動することができました。

```
.
├─ .fantasticonrc
├─ dist
|  └─ assets
|     └─ font
|        ├─ iconfont.css
|        ├─ iconfont.eot
|        ├─ iconfont.html
|        ├─ iconfont.woff
|        └─ iconfont.woff2
├─ package.json
└─ src
   ├─ css
   │  └─ foundation
   │     └─ iconfont.scss  // 移動されたSCSS
   └─ font
      ├─ svg
      │  ├─ blank.svg
      │  ├─ pdf.svg
      │  └─ ...
      └─ templates
         ├─ iconfont.html.hbs
         └─ iconfont.scss.hbs
```

# サンプル
https://github.com/sotalikefn/iconfont

# 参考
[fantasticon](https://www.npmjs.com/package/fantasticon)
[move-file-cli](https://www.npmjs.com/package/move-file-cli)
[自作したsvgアイコンをQuasarのQIconコンポーネントで表示する方法](https://zenn.dev/rescuenow/articles/f52ce4688c2647)
