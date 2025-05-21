# Qiita記事管理プロジェクト

このプロジェクトは、Qiita記事の作成と管理を効率的に行うためのワークスペースです。
プロジェクトの詳細な情報は[こちら](docs/about.md)をご覧ください。

## ディレクトリ構造

- `ideas/` - 記事のネタやアイデアをメモするディレクトリ
- `articles/` - 記事を管理するディレクトリ
  - `drafts/` - 下書き中の記事
  - `published/` - 公開済みの記事
  - `archived/` - アーカイブされた記事
- `docs/` - Qiitaへの記事投稿手順などのドキュメントを保存するディレクトリ

## 使い方

1. アイデアの記録
   - `ideas/` ディレクトリに新しいアイデアをマークダウンファイルとして保存
   - ファイル名は `YYYY-MM-DD_title-in-kebab-case.md` の形式を推奨

2. 記事の作成と管理
   - 新規記事は `articles/drafts/` に作成
   - 公開準備が整った記事は `articles/published/` に移動
   - 古くなった記事は `articles/archived/` に移動

3. Qiitaへの投稿
   - `docs/` ディレクトリの手順に従って記事を投稿
   - `articles/published/` 内の記事が自動的に公開されます 