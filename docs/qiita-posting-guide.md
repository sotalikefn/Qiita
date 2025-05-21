# Qiita投稿ガイド

## 準備

1. Qiitaアカウントの設定
   - [Qiita](https://qiita.com)にログイン
   - [設定] > [アプリケーション]から個人用アクセストークンを発行
   - トークンに必要な権限:
     - `read_qiita`
     - `write_qiita`

2. 設定ファイルの準備
   ```json
   // qiita.config.json
   {
     "accessToken": "your-access-token",
     "userName": "your-qiita-username"
   }
   ```

## 記事の投稿手順

1. 記事の作成
   - `articles/` ディレクトリに記事をマークダウンファイルで作成
   - フロントマターの例:
     ```markdown
     ---
     title: "記事のタイトル"
     tags:
       - JavaScript
       - React
     private: false  # true: 限定共有, false: 公開
     ---
     
     記事の本文...
     ```

2. 画像の添付
   - 画像は `articles/images/` ディレクトリに保存
   - マークダウンで参照: `![説明](./images/filename.png)`

3. プレビュー
   - VSCodeのマークダウンプレビュー機能を使用
   - または[Qiitaプレビュー](https://qiita.com/drafts/new)で確認

4. 投稿
   - Qiitaのウェブエディタにマークダウンをコピー＆ペースト
   - または[Qiita CLI](https://github.com/increments/qiita-cli)を使用して投稿

## 投稿後の管理

- 投稿した記事のURLを記事ファイルのフロントマターに追加
- 更新履歴をコミットメッセージとして記録

## トラブルシューティング

- アクセストークンの期限切れ
  - 新しいトークンを発行して `qiita.config.json` を更新
- 画像のアップロードエラー
  - ファイルサイズが10MB以下であることを確認
  - 対応フォーマット（PNG, JPG, GIF）であることを確認 