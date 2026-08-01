# Frontend

## 位置づけ

`matsu-front` は、利用者が `matsu` の機能を操作するためのブラウザ向けUIです。

アプリケーションAPIとJSON認証APIはBFFだけを呼び出します。`matsu-api`、`matsu-toolbox-api`、`matsu-arcade-api`、2つのAuth Serverへ直接業務requestを送りません。`matsu-auth` のlogin/register画面はBrowser redirectの遷移先ですが、開始とcallbackはBFFが管理します。

## 責務

- 画面表示、ユーザー操作、画面遷移
- 入力中の値など、クライアント側の一時状態管理
- BFFが公開する明示的なAPI routeの呼び出し
- BFF sessionとresource接続状態の確認
- 家計簿・Toolboxの認可開始とArcade login/register/disconnect操作の開始
- ローディング、入力error、API error、接続切れの表示

## 責務に含めないもの

- access token、refresh token、client secretの保存
- JWTの生成・検証やBearer headerの組み立て
- ユーザー資格情報の検証
- Backend APIやAuth ServerのJSON APIへの直接通信
- 各domainデータの永続化

## 概念構成

| 領域 | 役割 |
| --- | --- |
| Presentation | page、画面部品、style |
| Client State | 入力中の値や画面表示に必要な一時状態 |
| Server State | BFFから取得したデータ、cache、再取得 |
| BFF Client | BFF OpenAPIに基づく型付きAPI通信 |
| Authentication UI | session確認、接続開始、接続切れ通知 |

## 技術選定

| 技術 | この構成での狙い |
| --- | --- |
| React | component単位で画面と操作を構築する |
| TypeScript | UIとBFF API契約を型で接続する |
| Vite | 開発serverとFrontend buildを構成する |
| TanStack Query | BFFから取得するserver stateと再取得を管理する |
| OpenAPI | BFFを基準にrequest・response契約を共有する |
| `openapi-fetch` | 生成された型を利用してBFF routeを呼び出す |

## BFFとの境界

FrontendのAPI clientが持つBackend origin設定は `VITE_BFF_BASE_URL` だけです。通信ではBFFのsession Cookieを送るため `credentials: include` を使い、Browser JavaScriptからtokenを読み書きしません。

契約の流れは次のとおりです。

```text
BFF Route / Zod Schema
  -> BFF OpenAPI
  -> Frontend TypeScript Types
  -> openapi-fetch Client
```

BFFがresource別refreshを行った後も `401` となった場合、Frontendは対象機能の接続切れを扱います。家計簿向けには後方互換の明示refresh helperもありますが、Frontendがrefresh tokenを扱うことはありません。

FrontendはBFF OpenAPIから生成した型だけをサービス間schemaとして利用し、Resource APIのOpenAPIや内部型へ直接依存しません。

全体の正式経路は[システム全体構成](../architecture/system-overview.md)、tokenをBrowserから隠す仕組みは[認証・セッション構成](../architecture/authentication.md)、schemaの正本は[API契約](../architecture/api-contracts.md)を参照してください。
