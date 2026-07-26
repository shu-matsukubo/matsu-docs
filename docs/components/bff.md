# BFF

## 位置づけ

`matsu-bff`は、Frontend専用のBackend for Frontendです。

ブラウザとのセキュリティ境界としてセッションを管理し、Auth Serverに対してはOAuthクライアント、APIに対してはJWTを付与するクライアントとして動作します。また、Frontendに公開するAPI契約の所有者です。

## 責務

- `HttpOnly` Cookieを用いたブラウザセッションの管理
- アクセストークンとリフレッシュトークンのRedis保管
- Authorization Code + PKCEフローの開始とコールバック処理
- Confidential OAuth Clientとしてのコード交換とトークン更新
- Frontend向けの明示的なAPIルートとOpenAPI契約の提供
- APIへのBearer JWT付与
- リクエストとAPI成功レスポンスのスキーマ検証

## 責務に含めないもの

- ログイン・登録フォームの提供
- ユーザー資格情報の検証
- JWTへの署名
- 家計簿ドメインデータの永続化
- 他サービス向けの共有Redisの提供

## 概念構成

| 領域 | 役割 |
| --- | --- |
| Session Boundary | CookieとRedisセッションを対応付ける |
| OAuth Client | 認可開始、コールバック、コード交換、トークン更新 |
| Frontend API | Frontendに必要な明示的ルートを公開する |
| Contract | ZodスキーマとOpenAPIを管理する |
| Upstream Client | JWTを付けてAPIを呼び出し、レスポンスを検証する |

## 技術選定

| 技術 | この構成での狙い |
| --- | --- |
| TypeScript | Frontendと近い型システムでAPI契約とサーバー処理を記述する |
| Hono | 軽量なHTTPルーティングとミドルウェア構成を利用する |
| Zod | 実行時にリクエストとレスポンスを検証する |
| `@hono/zod-openapi` | ルート定義、検証スキーマ、OpenAPIを同じ契約から構築する |
| Redis | セッションと短命なOAuth状態をBFF境界内で管理する |
| Docker Compose | BFFと専用Redisを再現可能なローカル環境として起動する |

## セッション設計

ブラウザへは推測困難なセッションIDだけを渡し、Redis上で次の認証情報と対応付けます。

- アクセストークン
- リフレッシュトークン
- アクセストークンの有効期限

Cookie名は`matsu-session`です。Cookieは`HttpOnly`、`SameSite=Lax`を基本とし、HTTPS環境では`Secure`を有効にします。

認可処理中の`state`とPKCE verifierもRedisへ短期間保存します。通常のログインセッションとは用途と有効期間を分けて管理します。

## APIゲートウェイとしての方針

BFFは任意のAPIパスを透過的に中継するのではなく、Frontendに必要なルートを明示的に定義します。

- ブラウザ向け契約をBFF自身が所有する
- APIへ送るリクエストにアクセストークンを付与する
- APIの成功レスポンスをBFFのスキーマで検証する
- 契約に違反した上流レスポンスを、そのままFrontendへ渡さない

これにより、FrontendはAPI内部の変更から一定程度分離されます。

全体の通信関係は[システム全体構成](../architecture/system-overview.md)、認証処理は[認証・セッション構成](../architecture/authentication.md)を参照してください。
