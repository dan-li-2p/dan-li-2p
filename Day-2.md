## 🚀 開催概要

- 日程: 2024年10月29日(火) 10時30分～17時30分
- 場所
  - マイクロソフト本社 (品川)
  - Teams: [会議リンクはこちら](https://teams.microsoft.com/l/meetup-join/19%3ameeting_MWYwNTY0MDMtZGU1ZC00MjhiLTgyYTUtYjRkMjk5MmExYjY4%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%22e159a759-25b5-4aaf-96af-4e12f3d23747%22%7d)

## 事前準備

- [x] Static Web Apps の Easy Auth 設定 (Entra ID)
- [x] procedure.json の拡充

## アジェンダ

- [ ] バックエンドで認証情報が取れることの確認・Static Web Apps バックエンドリンク
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
  - フロントエンド・バックエンドを同一オリジンでホストできるため、Cookie ベースで認証が容易
  - SWA 経由以外からはバックエンドを呼び出せなくなる

### Search API の認証

- SWA バックエンドから Search API を呼び出す
  - 双方の App Service 間で VNET 統合を設定
  - API キー認証

## 参考ドキュメント

- [Azure Static Web Apps での API サポートの概要 | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/static-web-apps/apis-overview)