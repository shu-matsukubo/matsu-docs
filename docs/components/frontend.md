# Frontend

## 位置づけ

`matsu-front`は、利用者が家計簿機能を操作するためのブラウザ向けUIです。

データ取得と認証操作はBFFを経由し、APIやAuth ServerのJSON APIを直接呼び出しません。ログイン開始画面は持ちますが、資格情報を入力するログイン・登録フォームはAuth Serverが提供します。

## 責務

- 家計簿機能の画面表示とユーザー操作
- 画面遷移とクライアント側の状態管理
- BFFが公開するAPIの呼び出し
- BFFセッションの有効性確認
- 認証開始と、認証切れ時の再ログイン誘導
- ローディング、入力エラー、APIエラーの表示

## 責務に含めないもの

- アクセストークン、リフレッシュトークンの保存
- JWTの生成や検証
- ユーザー資格情報の検証
- 家計簿データの永続化
- APIとAuth Serverへの直接的な業務リクエスト

## 概念構成

| 領域 | 役割 |
| --- | --- |
| Presentation | ページ、画面部品、スタイル |
| Client State | 入力中の値や画面表示に必要な一時状態 |
| Server State | BFFから取得したデータ、キャッシュ、再取得 |
| BFF Client | OpenAPIに基づく型付きAPI通信 |
| Authentication UI | セッション確認、ログイン開始、認証切れ通知 |

## 技術選定

| 技術 | この構成での狙い |
| --- | --- |
| React | コンポーネント単位で画面と操作を構築する |
| TypeScript | UIとBFF API契約を型で接続する |
| Vite | 開発サーバーとFrontendビルドをシンプルに構成する |
| TanStack Query | BFFから取得するサーバー状態と再取得を管理する |
| OpenAPI | BFFを基準にリクエスト・レスポンス契約を共有する |
| `openapi-fetch` | 生成された型を利用してBFF APIを呼び出す |

## API契約

BFFが生成したOpenAPIドキュメントから、Frontend用のTypeScript型を生成します。URL、クエリ、リクエストボディ、成功・エラーレスポンスを手書きで重複定義しないことを基本方針とします。

契約の流れは次のとおりです。

```text
BFF Route / Zod Schema
  -> BFF OpenAPI
  -> Frontend TypeScript Types
  -> openapi-fetch Client
```

## 認証情報の扱い

- ブラウザが保持する認証関連情報は、BFFが発行する`HttpOnly`セッションCookie
- JavaScriptからアクセストークンやリフレッシュトークンを読み書きしない
- BFFから認証エラーの`401`を受けた場合はBFFのトークン更新を試みる
- 更新できなければセッション切れを通知し、ログインフローへ戻す

全体の流れは[認証・セッション構成](../architecture/authentication.md)を参照してください。
