## 🚀 開催概要

- 日程: 2024年10月22日(火) 10時30分～17時30分
- 場所
  - マイクロソフト本社 (品川)
  - Teams: 午前 [会議リンクはこちら](https://teams.microsoft.com/l/meetup-join/19%3ameeting_NjAzMjFiYzctMTk5ZS00ZWZmLWJjYzEtOWU2ZWE0NzEzODRj%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2267f64f6a-9fe9-4603-bbfa-ce3ad1558762%22%7d) / 午後 [会議リンクはこちら](https://teams.microsoft.com/l/meetup-join/19%3ameeting_YzBhZGY4YWMtOWQ2Ni00ZTZiLTg5NjgtMmU3M2Y4Y2E1MTQz%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2267f64f6a-9fe9-4603-bbfa-ce3ad1558762%22%7d)

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
- ターミナルで `npm run dev` を実行してローカルで Nuxt を起動
  - コマンドを実行するとサーバーが立ち上がるので、ブラウザでアクセスする
  - Codespaces の通知が表示されるため、そこからブラウザーで開くを選択すると簡単に表示できる
  - "Welcome to Nuxt" のページが表示されていれば成功
  - `App.vue` にページの実体が存在している

### ✅ Nuxt に新しいページを追加する

- `frontend` ディレクトリの直下に `pages` ディレクトリを作成
- `pages` ディレクトリに `index.vue` ファイルを作成
  - `<template>` タグを追加して、その中に更に適当に `<div>` タグなどを追加する
  - 保存するとホットリロードで自動的に反映される
- `App.vue` ファイルを修正して、ページを読み込むようにする
  - `<div>` の中に存在しているデフォルト定義を全て削除し、`<NuxtPage />` タグを追加
  - ホットリロード後にページの再読み込みを行うと `index.vue` が表示される
- ページの下側に表示されている Nuxt のアイコンを選択すると Nuxt DevTools が起動する
  - WebSocket が必要になるのでネットワークの制限によっては動作しない
  - 可能な限り VS Code Desktop で開くことをお勧め
- Nuxt DevTools を使うと現在のページ、コンポーネントの状態を確認できる
  - 初回起動時に Authorize が必要になるため、デバッグコンソールに表示されたトークンをコピーして入力
  - コンポーネントの `props` を確認できるのはデバッグ時に非常に便利、お勧め
  - ESLint Config Inspector を使うと現在のルールや適用されているルールを確認できる
  - Payload を使うと、サーバーからダウンロードしたデータを Key-Value 形式で確認できる

### ✅ Nuxt のモードを SPA に変更する

- Static Web Apps にデプロイするには SPA モードが必須
- `nuxt.config.ts` に `ssr: false` を追加する
  - デフォルトの SSR モードでは CI 中にもデータを取ってレンダリングまで行おうとする
  - 認証が必要な場合は SSR を使うことは難しい（実質無理）

### ✅ Codespaces に GitHub Copilot をインストール

- Codespaces サイドバーの拡張機能から Copilot を検索
  - `GitHub Copilot` を選択してインストール
- インストールした GitHub Copilot 拡張を `devcontainer.json` に追加
  - 右クリックメニューにある `Add devcontainer.json` を選択する

### ✅ VS Code Desktop で Codespaces を開く

- VS Code Desktop を立ち上げて、Codespaces 拡張機能をインストールする
- サイドバーに増えた Remote Development を開いて GitHub へログイン
  - ブラウザが立ち上がって許可が求められるので、緑色のボタンを押して許可
- ログインが完了すると起動中 Codespaces の一覧が表示されるので、🔌 アイコンをクリックして接続

### ✅ Azure にリソースを作成

- サブスクリプションにリソースプロバイダーが登録されているか確認
  - `Microsoft.Web` と `Microsoft.Storage` の 2 つを登録
- リソースグループを最初に作成
  - リージョンは基本 `Japan East` を選択
- Static Web App を作成
  - API リンクを使うので Standard が必須
  - GitHub リポジトリを選択するためにログインを行う
  - リポジトリを選択し、ビルド設定を行う
- SWA のリソースを作成すると、GitHub 側に yml ファイルが作成される
  - yml は GitHub Actions の定義ファイルで、ビルドなどの処理が定義されている
  - 自動生成された yml では `npm run generate` が使われていないので修正が必要
  - Node.js バージョンも古いので修正する

### ✅ UI ライブラリの選定

- RXC と合わせる形で Nuxt UI (Tailwind ベース) を選定
  - Pro を利用するかどうかは要検討
  - 今回の画面構成では必要ない可能性が高い

### ✅ 手順書詳細のページを実装する

- Nuxt UI を nuxi を使ってインストールする
  - `npm nuxi@3.13.2 module add ui` を実行
  - `nuxt.config.ts` に `@nuxt/ui` が追加されていることを確認
- `pages` ディレクトリの中に `procedures/[id].vue` ファイルを作成
  - 作成したファイルのルートに `<template>` タグを追加
  - 更に `<template>` タグの中に `<div>` と `<UButton>` タグを追加
- データモデルの仕様を検討
  - 工程毎にオブジェクトを定義する
  - ステータス、最終更新日、最終更新者は全ての工程で必須
- 固定のデータを定義してレンダリングの処理を定義
  - Tailwind を使ってパディングなどを実装

### ✅ GitHub Actions を使ったデプロイを実装

- SPA として動作させるために `staticwebapp.config.json` を追加
  - `navigationFallback` を設定し、存在しないパスへのリクエスト時にも `index.html` を返す
- Pull Request を作成して、プレビュー環境でレビュー
  - ブランチを切って開発をするとレビュー中に変更点を確認しやすくなる
- Environment を作成してデプロイ前レビューを必須にする
  - レビューが必須かどうかは Environment 単位で指定が可能
  - ワークフロー定義でどの Environment を使うかを明記する
- 本来であれば Environment 単位でワークフローやジョブを分けていただくのがベスト

### ✅ サンプルデータ保存用のストレージ、JSON を作成

- ストレージアカウントをデプロイ
  - LRS を指定してデプロイしていただく
- サンプルに使用したデータを JSON ファイルにそのまま変換する
  - Codespaces で作成したファイルを右クリックメニューでダウンロード
- ストレージアカウントに匿名アクセス可能なコンテナーを作成
  - 構成から匿名アクセスを有効化する必要がある
- SPA からアクセスできるように CORS の設定を追加
  - 許可されたオリジン、ヘッダーには `*` を設定

## 参考ドキュメント

- [Nuxt: The Intuitive Vue Framework · Nuxt](https://nuxt.com/)
- [開発コンテナーの概要 - GitHub Docs](https://docs.github.com/ja/codespaces/setting-up-your-project-for-codespaces/adding-a-dev-container-configuration/introduction-to-dev-containers)
- [Rendering Modes · Nuxt Concepts](https://nuxt.com/docs/guide/concepts/rendering)
- [Nuxt UI: A UI Library for Modern Web Apps](https://ui.nuxt.com/)
- [IntelliSense - Installation - Nuxt UI](https://ui.nuxt.com/getting-started/installation#intellisense)
- [GitHub Actions ドキュメント - GitHub Docs](https://docs.github.com/ja/actions)
- [フォールバック ルート - Azure Static Web Apps を構成する | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/static-web-apps/configuration#fallback-routes)
- [Azure Static Web Apps で環境をプレビューする | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/static-web-apps/preview-environments)
- [デプロイに環境の使用 - GitHub Docs](https://docs.github.com/ja/actions/managing-workflow-runs-and-deployments/managing-deployments/managing-environments-for-deployment)