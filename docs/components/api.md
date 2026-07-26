# API

## 位置づけ

`matsu-api`は、家計簿ドメインの処理と永続データを管理するBackend APIです。

ブラウザから直接利用することを前提とせず、BFFからBearer JWT付きで呼び出されます。認証情報の発行やブラウザセッションではなく、認証済みユーザーに対するドメイン処理に責任を持ちます。

## 責務

- 支出、支払方法、カテゴリなどの家計簿データの管理
- 家計簿ドメインの入力検証と処理
- MySQLへの永続化と検索
- APIレスポンスの生成
- Auth Serverが発行したJWTの検証
- 認証されたユーザーを基準としたデータアクセス

## 責務に含めないもの

- ログイン・ユーザー登録画面の提供
- ユーザー資格情報の検証
- アクセストークンやリフレッシュトークンの発行
- ブラウザのCookieとセッションの管理
- Frontend固有の状態管理

## 概念構成

| 領域 | 役割 |
| --- | --- |
| HTTP Boundary | ルート、入力検証、レスポンス形式 |
| Authentication | JWTを検証し、リクエスト主体を特定する |
| Application / Domain | 家計簿のユースケースと業務処理 |
| Query / Persistence | データ検索とMySQLへの永続化 |

## 技術選定

| 技術 | この構成での狙い |
| --- | --- |
| PHP | Laravelエコシステム上でBackend APIを実装する |
| Laravel | ルーティング、検証、DI、ORM、マイグレーションなどを一貫して利用する |
| MySQL | 家計簿のリレーショナルデータを永続化する |
| JWT | BFFからのリクエストをステートレスに認証する |
| JWKS | Auth Serverの秘密鍵を共有せず、公開鍵でJWT署名を検証する |
| Docker Compose | PHP/ApacheとMySQLを再現可能なローカル環境として起動する |

## 認証方式

APIはAuth Serverが発行したRS256 JWTアクセストークンを受け取ります。

JWT検証では、少なくとも次の要素を信頼判断に使用します。

- Auth Serverによる署名
- issuer
- audience（`matsu-api`）
- トークンの有効期限

署名検証用の公開鍵はAuth ServerのJWKSエンドポイントから取得し、Laravelのキャッシュを利用して保持します。APIはAuth Serverの秘密鍵を持たず、JWTの発行も行いません。

## BFFとの境界

- BFFはブラウザセッションをアクセストークンへ変換する
- APIはBearer JWTを検証してドメイン処理を実行する
- BFFはAPIレスポンスをFrontend向け契約として検証する
- APIはFrontendのCookieや画面都合を認識しない

この分担により、APIはブラウザ固有の認証状態から分離され、家計簿ドメインに集中します。

全体の通信関係は[システム全体構成](../architecture/system-overview.md)、JWTの受け渡しは[認証・セッション構成](../architecture/authentication.md)を参照してください。
