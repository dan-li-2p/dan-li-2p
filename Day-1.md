## 🚀 開催概要

- 日程: 2024年10月22日(火) 10時30分～17時30分
- 場所
  - マイクロソフト本社 (品川)
  - Teams: [会議リンクはこちら](https://teams.microsoft.com/l/meetup-join/19%3ameeting_NjAzMjFiYzctMTk5ZS00ZWZmLWJjYzEtOWU2ZWE0NzEzODRj%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2267f64f6a-9fe9-4603-bbfa-ce3ad1558762%22%7d)

## 🎯️ ゴール

- ASP.NET MVC で構成されている既存システムのカスタマイズ版を Nuxt/Vue で Azure 上に構築

## 💭 ディスカッション

- 3 月のリリースに向けての疑問点の解消を最優先にする
  - Nuxt を使って開発する
- Day 1 は Nuxt の基本的な開発、デプロイを目標とする
- Day 2 ではデータストアを RDB から NoSQL に出来ないかを検討する（予定）

## 🔖 作業ログ

### ✅ Nuxt の開発環境を構築

- GitHub Codespaces を利用して Nuxt + Vue の環境を立ち上げる
  - VS Code の拡張機能（ESLint / Volar など）が必須
  - 定義済みの Dev Container 定義を利用する
- `.devcontainer` ディレクトリ以下に `devcontainer.json` を作成して定義を置いておく