## 🚀 開催概要

- 日程: 2024年10月29日(火) 10時30分～17時30分
- 場所
  - マイクロソフト本社 (品川)
  - Teams: [会議リンクはこちら](https://teams.microsoft.com/l/meetup-join/19%3ameeting_MWYwNTY0MDMtZGU1ZC00MjhiLTgyYTUtYjRkMjk5MmExYjY4%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%22e159a759-25b5-4aaf-96af-4e12f3d23747%22%7d)

## 事前準備

- [x] Static Web Apps の Easy Auth 設定 (Entra ID)
- [x] procedure.json の拡充

## アジェンダ

- [x] バックエンド API と Search API の VNET 統合
- [x] バックエンド API で認証情報が取れることの確認・Static Web Apps バックエンドリンク
  - バックエンドと Search API との認証方法
- [x] フォーム画面実装
- [x] 開発環境/ステージングスロットについて (アクセス制限等)
  - プレビュー環境用に Entra ID の応答 URL をどう設定すれば良いか
- [ ] [Optional] データ設計のディスカッション

## ディスカッション

### SWA バックエンドリンクとは？

- SWA では基本静的アセットしかホストできない。バックエンド (API) のホストはオプション機能で、認証設定の統合が可能
  - 標準ではマネージド Function (Azure Functions)
  - バックエンドリンクで任意の App Service/Azure Functions/API Management/Container Apps リソースとリンクできる
- バックエンドリンクを設定すると `/api` へのアクセスがリンクしたバックエンドにリバースプロキシされる
  - フロントエンド・バックエンド API を同一オリジンでホストできるため、Cookie ベースで認証が容易
  - SWA 経由以外からはバックエンドを呼び出せなくなる

### Search API/ファイルアップロードツールの認証

- SWA バックエンドから Search API を呼び出す
  - 双方の App Service 間で VNET 統合を設定
  - API キー認証
  - ログインユーザーの識別はパラメータで oid を引き渡して行う
- ファイルアップロードツールはメンテナンスツールなので、VNET 内に配置

### アーキテクチャ

<img width="700" alt="architecture" src="https://github.com/user-attachments/assets/e0888884-1c80-45fa-81cd-4ef0e1546be6">

### 環境の分け方について

- 開発・ステージング・本番用にそれぞれリソースを分ける
- スロットは基本的に本番リリース直前の swap 用途のみで使う
- 性能試験はステージングで行う → ステージングと本番のプランを揃えておく必要がある
- ステージング環境を使わない期間はどうするか？
  - 無効化は PaaS のリソース設定上できないので、コスト削減したいときはスケールダウン or リソース削除
    - App Service は B1 まで落とせる (VNET 統合可能)
    - IaC を導入しておくとリソース再作成がやりやすい
- SWA プレビュー環境は環境ごとにオリジンが変わるが、認証用の Entra ID アプリの応答 URL はどう設定する？
  - 応答 URL にアスタリスクを設定できるが、通常 UI ではバリデーションが通らないため、マニフェスト編集画面で操作する必要がある

### ローカル開発時の認証

- SWA CLI で認証をエミュレーションできる

### DB 選定

- 手順書データ・設備詳細情報のネストが複雑なので、RDB よりも Cosmos DB (NoSQL) の方が API スキーマとのマッピング実装が楽
- Cosmos DB を使う場合の注意点
  - 集計系のデータは Change Feed でマテリアライズドビュー化する (RDB or Cosmos) 
  - インデックスがデフォルトで全項目に設定されるので、必要な項目のみに絞る
  - マスタデータとの紐付けは、マスタ更新頻度が低い場合は各データ内に埋め込みでも OK。更新頻度が高ければ外部参照にしても良い

## 🔖 作業ログ

### ✅ バックエンド API と Search API の VNET 統合

Azure ポータルで各リソースを作成・設定

- VNET (仮想ネットワーク) を作成
  - リージョンは Japan East に設定
  - IP アドレスタブで App Service 用のサブネットを作成
    - アドレス範囲: `10.0.0.0/16`
    - サイズ: `/24` (App Service を入れるには最低 27 bit 必要)
- App Service (バックエンド API) を作成
  - スタックは .NET 8 を選択
  - OS は Linux を選択
  - リージョンは Japan East に設定 (VNET と同様にする必要あり)
  - App Service プラン (バックエンド API & Search API 共有用) を P0V3 で新規作成
    - ゾーン冗長は無効に設定 (本番環境では SLA 上必要であれば有効化)
  - ネットワーク設定はひとまずデフォルトにしておく
  - Application Insights は無効に設定 (運用時には有効化)
  - Microsoft Defender for Cloud は無効に設定 (コスパが良くないので基本無効で OK)
- App Service (バックエンド API) のネットワーク画面で送信トラフィックの制限を設定
  - 送信トラフィックの構成 > 仮想ネットワーク統合 からVNET 統合を追加
    - 先ほど作成した VNET とサブネットを選択
    - 送信インターネットトラフィックを有効化 (Outbound トラフィックをすべて VNET 経由に制限)
- App Service (Search API) を作成
  - 初期設定はバックエンド API と同様
  - App Service プランはバックエンド API と共有するよう設定
- App Service (Search API) のネットワーク画面で受信トラフィックの制限を設定
  - 公衆ネットワークアクセスの設定でアクセス制限を追加
    - 選択した仮想ネットワークと IP アドレスから設定 を選択
    - サイトのアクセスとルールを設定
      - 一致しないルールのアクションは「拒否」に設定
      - Firewall ルールを追加
        - 優先度を 100 に設定
        - ソースの設定で仮想ネットワークを選択し、先ほど作成した VNET とサブネットを選択
- App Service (Search API) の URL に VNET 外でブラウザからアクセスすると期待通り 403 エラーになることを確認

### ✅ Static Web Apps バックエンドリンク

Azure ポータルで Static Web Apps に バックエンド API の App Service をリンク

- Static Web Apps > API 画面で Production 環境のリンクを追加
  - バックエンドリソースの種類は Web アプリを選択
  - リソース名で バックエンド API の App Service リソースを選択
- App Service (バックエンド API) の URL にブラウザからアクセスすると期待通り 400 エラーになることを確認

### ✅ Nuxt に関する Q&A

- 詳細と一覧を実装
  - Nuxt UI を使ってコンポーネントを実装
  - 表示用の JSON を利用して表示している
- フッターの扱いが難しい
  - Flex を使うと綺麗に作れる
  - Tailwind CSS のプリセットを使っても作れる
- トップページにアクセスした場合に一覧にリダイレクトしたい
  - SPA ではリダイレクトは難しい
  - Nuxt のミドルウェアを使うとページ表示前に処理を追加できる
- Nuxt コンポーネントのカスタマイズ
  - `ui` prop にオブジェクトを渡すことでオーバーライドが可能
  - Nuxt UI のリファレンスにある Config を見るとデフォルトで `ui` prop に当たっている設定を確認できる
- Nuxt UI に `UIcon` コンポーネントがある
  - Material Icons なども組み込みで利用できる
  - 利用する Icon Set のみインストールするようにするとバンドルサイズを削減できる
  - 色を変えるのは `UIcon` の `class` で変更
  - Vue の場合は `:class` を使うと動的にバインディング可能
- Nuxt のミドルウェアを利用してリダイレクトを行う
  - `middleware` ディレクトリを作成する
  - 強制的にリダイレクトを行うミドルウェアを作成
  - 該当のページで `definePageMeta` を使って利用するミドルウェアを参照する
- フッターを Flex を使って実現する
  - 親の `div` に対して `class=flex flex-col h-screen` を追加して縦積みを指定
  - `slot` には `class` を指定できないので `main` で囲んで `class=grow` を指定

### ✅ Nuxt を利用してフォームを実装

- ページコンポーネントで `useFetch` を行うか、コンポーネントで `useFetch` を行って props で渡すか
  - コンポーネントのキャッシュがあるので、最近はページコンポーネントで `useFetch` を行う方が多い
- `useFetch` で取ってきたデータをそのまま渡して書き換えるのではなく、更新用のデータを用意する
  - `ref` を使って `useFetch` で取ってきたデータを書き換え可能にする
  - `ref` には初期値が必要なので一旦 `undefined` を指定する
  - `ref<Procedure | undefined>` のような Union 型を指定して警告を消す
- 本来はページコンポーネントで `useFetch` を行った方が、コンポーネントでは値がある前提で進められてわかりやすい
  - `undefined` 以外であることを保証するために `v-if` で条件を指定する
- 表示のみ必要な場合は `procedure` を参照する
  - `updateBy` や `updateAt` など
- 「保存」ボタンが押された時の処理を追加する
  - `@click` を指定して呼び出される処理を実装する
  - コンソールにログを出力する処理だけを追加
- 親のページコンポーネント側で `useFetch` を行うように変更する
  - `Procedure` の型定義を別ファイルに切り出す
  - `defineProps` を呼び出して、親から `Procedure` を受け取れるようにする
- 「保存」ボタンが押された時に API を呼び出す処理を実装する
  - イベントハンドラーでは `use***` 系の API は利用できない
  - 代わりに `$fetch` を使って `/api/procedures/{id}` への PUT リクエストを実装する
  - PUT リクエストの `body` にデータ全体を渡すために `procedureForm.value` を渡す
- 保存完了後にページの情報を読み直す処理を実装する
  - コンポーネントからページコンポーネントへイベントをエミットする形で実装する
  - `defineEmits` でエミットするイベントを定義する
  - PUT リクエストの発行後に定義したイベントをエミットする
  - エミットされたイベントはコンポーネントの属性としてイベントハンドラーで受け取る

### ✅ SWA CLI でローカル環境での認証エミュレーション

SWA CLI を Codespace にインストール

- Dev Container の postCreateCommand に SWA CLI インストールコマンドを追加
- コンテナをリビルド後、VS Code ターミナルで swa コマンドが実行できることを確認

バックエンドコードを追加

- コマンドパレット > `Codespaces: Add Dev Container Configuration Files...` から `Modify your active configuration` を選択し、Dotnet CLI を選択
- コンテナをリビルド後、VS Code ターミナルで dotnet コマンドが実行できることを確認
- backend ディレクトリを追加し、`dotnet new webapi` コマンドを実行し Web API ボイラープレートコードを生成
- `dotnet run` コマンドでローカル起動し、`/weatherforecast` エンドポイントにアクセスできることを確認
- `/weatherforecast` の path を `/api/weatherforecast` に変更しておく

SWA CLI でフロントエンド・バックエンドをローカル起動

- `dotnet run` コマンドでバックエンドを起動しておく
- `swa init` コマンドで SWA CLI 用の設定ファイル (swa-cli.config.json) を生成
  - appDevserverUrl のポートを Nuxt Dev サーバーのポート (3000) に変更
  - apiDevserverUrl に .NET バックエンドの URL を設定
- `swa start` コマンドで 4280 ポートが起動し、認証エミュレーターの画面が開くことを確認
- 適当な Username を入力してログインすると Nuxt アプリ画面が開き、`<SWA CLI エミュレータのオリジン>/api/weatherforecast` にアクセスできることを確認

バックエンドで PUT エンドポイントを実装

- `app.MapGet` を `app.MapPut` に変更し、path を `/api/weatherforecast` から `/api/procedures/{id}` に変更
- 引数で id と body を受け取りレスポンスで返すよう実装
- 詳細画面を開き保存ボタンを押下すると、PUT リクエストが成功してレスポンスで値が返ることを確認

バックエンド側での認証情報取得を試行

- API 関数の引数に HTTPContext 型の context を追加
- [ドキュメント](https://learn.microsoft.com/ja-jp/azure/static-web-apps/user-information) のサンプルコード通りに HTTP リクエストヘッダーの `x-ms-client-principal` の内容をパースするよう実装
- パースした内容もレスポンスで返すよう実装
- 詳細画面を開き保存ボタンを押下すると、PUT リクエストが失敗し JSON シリアライズエラーが出力されることを確認
- レスポンスで認証情報の名前のみを返すように修正し再試行すると、PUT リクエストが成功し値が返ることを確認

## 参考ドキュメント

- [Azure Static Web Apps での API サポートの概要 | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/static-web-apps/apis-overview)
- [アプリを Azure 仮想ネットワークと統合する - Azure App Service | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/app-service/overview-vnet-integration)
- [Icon - Nuxt UI](https://ui.nuxt.com/components/icon)
- [Config | Card - Nuxt UI](https://ui.nuxt.com/components/card#config)
- [middleware/ · Nuxt Directory Structure](https://nuxt.com/docs/guide/directory-structure/middleware)
- [plugins/ · Nuxt Directory Structure](https://nuxt.com/docs/guide/directory-structure/plugins)
- [Documentation | Terraform | HashiCorp Developer](https://developer.hashicorp.com/terraform)
- [Azure Static Web Apps のプレビュー環境で Microsoft Entra ID 認証を有効化する - しばやん雑記](https://blog.shibayan.jp/entry/20240108/1704679165)
- [独自のバックエンドとリンクした Azure Static Web Apps のプレビュー環境を自動で作成する - しばやん雑記](https://blog.shibayan.jp/entry/20240302/1709354966)
- [Static Web Apps CLI Documentation | Static Web Apps CLI](https://azure.github.io/static-web-apps-cli/)
- [Azure Static Web Apps でのユーザー情報へのアクセス | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/static-web-apps/user-information)