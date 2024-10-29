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
- [ ] フォーム画面実装
- [ ] 開発環境について (アクセス制限等)
  - プレビュー環境用に Entra ID の応答 URL をどう設定すれば良いか
- [ ] ステージングスロットについて
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

<img width="700" alt="architecture" src="https://github.com/user-attachments/assets/f21261ec-84c2-4e43-8d4d-9cee33791d0c">

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

### ✅ Nuxt

- 詳細と一覧を実装
  - Nuxt UI を使ってコンポーネントを実装
  - 表示用の JSON を利用して表示している
- フッターの扱いが難しい
  - Flex を使うと綺麗に作れる
  - Tailwind CSS のプリセットを使っても作れる
- SPA ではリダイレクトは難しい
  - Nuxt のミドルウェアを使うとページ表示前に処理を追加できる
- Nuxt コンポーネントのカスタマイズ
  - `ui` prop にオブジェクトを渡すことでオーバーライドが可能
  - Nuxt UI のリファレンスにある Config を見るとデフォルトで `ui` prop に当たっている設定を確認できる

## 参考ドキュメント

- [Azure Static Web Apps での API サポートの概要 | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/static-web-apps/apis-overview)
- [アプリを Azure 仮想ネットワークと統合する - Azure App Service | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/app-service/overview-vnet-integration)