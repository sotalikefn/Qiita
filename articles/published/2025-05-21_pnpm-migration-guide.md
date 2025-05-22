---
title: nodebrewユーザーのためのnpmからpnpmへの移行ガイド
tags:
  - npm
  - pnpm
  - nodebrew
  - Node.js
  - パッケージマネージャー
private: true
updated_at: '2025-05-21T10:00:00+09:00'
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

JavaScriptプロジェクトの開発において、パッケージマネージャーは欠かせないツールです。長らく`npm`がデファクトスタンダードとして使われてきましたが、近年は`pnpm`のような代替ツールが注目を集めています。

本記事では、`npm`から`pnpm`への移行を検討している開発者、特に`nodebrew`でNode.jsのバージョン管理をしている方に向けて、移行のメリットと具体的な設定手順を解説します。

## 前提条件

- Node.js（v14以上）がインストールされていること
- npmの基本的な使い方を理解していること
- nodebrewを使用している場合は、最新版にアップデートしておくこと

## 本題

### pnpmとは何か？その特徴と利点

pnpm（pronounced "p-n-p-m"）は、高速でディスク効率の良いパッケージマネージャーです。従来のnpmやyarnと比較して、以下のような特徴があります。

#### 🔥 pnpmが優れているポイント

| 比較項目            | pnpm                                  | npm                                |
|---------------------|---------------------------------------|-------------------------------------|
| インストール速度     | ✅ 速い（ハードリンク使用）            | 遅め                                |
| ディスク使用量       | ✅ 少ない（依存を共有）                | 多い（プロジェクトごとに重複）      |
| 依存の厳格性         | ✅ 厳格（バグの早期発見につながる）     | 緩い                                |
| キャッシュ管理       | ✅ 賢い                                | 普通                                |
| 互換性               | 一部ツールで注意が必要                | ✅ 最も広くサポートされている        |

pnpmの最大の特徴は、同じバージョンのパッケージを複数のプロジェクトで共有するという点です。これにより：

1. **ディスク容量の大幅な節約**：同じパッケージが重複してインストールされません
2. **インストール速度の向上**：キャッシュとハードリンクを活用した高速なインストール
3. **厳格な依存関係管理**：隠れた依存関係を防ぎ、バグを早期に発見

#### 👎 pnpmの注意点

完璧なツールはありません。pnpmにも以下のような注意点があります：

- 一部のツールやスクリプトがpnpm非対応の場合がある
- チームで統一して使用する必要がある（混在すると問題が生じる可能性）
- 従来のnpmのワークフローに慣れている場合、少し学習コストがかかる

### npmとpnpmの動作比較

#### 依存関係の解決方法の違い

npmとpnpmでは、依存関係の解決方法に大きな違いがあります。

**npmの場合**：
- 各プロジェクトごとに `node_modules` ディレクトリにすべての依存パッケージをフラットにインストール
- ディスク使用量が多くなりがち
- 「ファントム依存関係」問題（明示的に依存関係を宣言していなくても利用できてしまう）が発生

**pnpmの場合**：
- 依存パッケージはグローバルストアに一度だけインストールされ、プロジェクト間で共有
- `node_modules` 内には実際のファイルではなくシンボリックリンクを配置
- 「ファントム依存関係」問題を回避する厳格な依存関係管理

#### ロックファイルの違い

```
# npmの場合
package-lock.json

# pnpmの場合
pnpm-lock.yaml
```

どちらも依存関係を正確に再現するためのファイルですが、形式と内部構造が異なります。

### nodebrewユーザー向けのセットアップ手順

nodebrewでNode.jsのバージョン管理をしている方に向けて、pnpmのセットアップ手順を解説します。
結論から申し上げると、Node.js v16.10以降をお使いであれば、pnpmをインストールする必要は無く、有効化するコマンドを実行することでpnpmが使用可能となります。

### セットアップコマンド
Node.jsに組み込まれている「corepack」を活用します。

#### 🧭 corepackとは？

corepackは、Node.js v16.10以降に組み込まれているツールで、様々なパッケージマネージャー（pnpm, Yarn, npm）を統一的に管理できます。

- corepackもNode.js本体に組み込まれているため、別途インストールが不要
- パッケージマネージャーのバージョン管理と自動実行が可能
- プロジェクトごとに使用するパッケージマネージャーのバージョンを固定できる

#### 🔧 基本的なセットアップ手順

```bash
# 1. corepackを有効化する（Node.jsのインストール後、1回だけ必要）
corepack enable

# 2. 任意のバージョンのpnpmを使用可能にする
corepack prepare pnpm@8.15.4 --activate

# 3. プロジェクトのpackage.jsonにパッケージマネージャーを指定
# package.jsonに以下を追加
{
  "packageManager": "pnpm@8.15.4"
}
```

#### ⚠️ nodebrewでの注意点

:::note alert
nodebrewでNode.jsのバージョンを切り替えた場合、**各バージョンで初回の1回だけ** `corepack enable` を実行する必要があります
:::

```bash
# 例：Node.jsのバージョンを切り替えた場合
nodebrew use v18.16.0
corepack enable  # このNode.jsバージョンでcorepackを有効化
```


### pnpmの基本的な使い方

npmを使っていた方がpnpmに移行する際の、基本的なコマンドの対応表です。

| 操作                | npm                    | pnpm                   |
|---------------------|------------------------|------------------------|
| インストール         | `npm install`          | `pnpm install`         |
| パッケージの追加     | `npm install パッケージ` | `pnpm add パッケージ`   |
| 開発用パッケージの追加 | `npm install -D パッケージ` | `pnpm add -D パッケージ` |
| グローバルインストール | `npm install -g パッケージ` | `pnpm add -g パッケージ` |
| スクリプト実行       | `npm run スクリプト名`   | `pnpm スクリプト名`     |
| パッケージの削除     | `npm uninstall パッケージ` | `pnpm remove パッケージ` |
| 依存関係の更新       | `npm update`          | `pnpm update`         |

特に注意すべき点は、`pnpm`ではスクリプト実行時に`run`を省略できることです。これにより、日常的なコマンド実行がより簡潔になります。

### pnpmへの移行手順（既存プロジェクト）

既存のnpmプロジェクトをpnpmに移行する手順を紹介します。

```bash
# 1. corepackを有効化（初回のみ）
corepack enable

# 2. 希望するpnpmバージョンを準備
corepack prepare pnpm@8.15.4 --activate

# 3. node_modulesとpackage-lock.jsonを削除
rm -rf node_modules package-lock.json

# 4. pnpmでインストールを実行
pnpm install

# 5. package.jsonにパッケージマネージャー情報を追加
# 以下をpackage.jsonに追加
# "packageManager": "pnpm@8.15.4"
```

これで、プロジェクトがpnpmを使用するように変更されます。`pnpm-lock.yaml`が生成され、依存関係が再構築されます。

## まとめ

npmからpnpmへの移行は、以下のようなメリットをもたらします：

- ディスク使用量の削減
- インストール速度の向上
- より厳格な依存関係管理

特にnodebrewユーザーは、corepackを活用することで、Node.jsのバージョン管理とpnpmの管理を効率的に行うことができます。

最初は新しいコマンドや概念に慣れる必要がありますが、一度使い始めると、その効率性と一貫性から開発体験が向上するでしょう。

## 参考資料

- [pnpm公式ドキュメント](https://pnpm.io/)
- [Node.js Corepack documentation](https://nodejs.org/api/corepack.html)
- [pnpm vs npm vs Yarn比較記事](https://blog.logrocket.com/javascript-package-managers-compared/)
- [Corepackを使ったパッケージマネージャーの管理](https://dev.to/azure/what-is-corepack-and-why-it-is-so-useful-467a)