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
  - プロジェクト全体で利用するフォーマット、規約を強制できる
  - `devcontainer.json` をコミット、プッシュしてから開き直す
- "New with options..." を選択して、作成した Dev Container 定義（Node.js 20）が選ばれていることを確認
  - Machine type を 4-core に変更
- ブラウザ上で開発するだけではなく、ローカルにインストールされた VS Code で開くこともできる
  - Codespaces のハンバーガーメニューから "Open in VS Code Desktop" を選択
  - ローカルに VS Code のインストールが必須
- Nuxt のインストールを実行する
  - [Installation · Get Started with Nuxt](https://nuxt.com/docs/getting-started/installation) のコマンドを実行
  - `frontend` というディレクトリを切ってプロジェクトを作成
  - パッケージマネージャは `npm` を選択するとボイラープレートが作成される
  - 初回は `npm install` が実行されるので時間がかかる
- Nuxt ESLint をインストールする
  - [ESLint - Code Style · Nuxt Concepts](https://nuxt.com/docs/guide/concepts/code-style#eslint)
  - 最新の nuxi にバグがあるため `npx nuxi@3.13.2 module add eslint` を実行
  - 実行後 `nuxt.config.ts` に `@nuxt/eslint` が追加されていることを確認
- ESLint の追加設定を行う
  - `nuxt.config.ts` で `eslint` のオプションを追加
  - `eslint` => `config` 以下に `stylistic: true` を追加
- VS Code 上での Nuxt の type error を修正する
  - `npx nuxt prepare` を実行すると型情報が再生成されてエラーが消える
- ファイル保存時に自動的に整形する設定を追加
  - `.vscode` ディレクトリをルートに作成し、中に `settings.json` ファイルを作成
  - `editor.formatOnSave` キーを追加
  - ESLint を使ってフォーマットを行う設定が追加されている
  - 設定後には保存時に ESLint がフォーマットを行ってくれる

### ✅ Codespaces に GitHub Copilot をインストール

- Codespaces サイドバーの拡張機能から Copilot を検索
  - `GitHub Copilot` を選択してインストール
- インストールした GitHub Copilot 拡張を `devcontainer.json` に追加
  - 右クリックメニューにある `Add devcontainer.json` を選択する

## 参考ドキュメント

- [Nuxt: The Intuitive Vue Framework · Nuxt](https://nuxt.com/)
- [開発コンテナーの概要 - GitHub Docs](https://docs.github.com/ja/codespaces/setting-up-your-project-for-codespaces/adding-a-dev-container-configuration/introduction-to-dev-containers)