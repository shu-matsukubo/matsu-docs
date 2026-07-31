# BFF

## 位置づけ

`matsu-bff` は、`matsu-front` 専用のBackend for Frontendです。

Browserとのsecurity boundaryとしてsessionを管理し、2つのAuth Serverとの認証処理を仲介し、3つのResource APIへ対応するBearer JWTを付けて呼び分けます。汎用reverse proxyではなく、Frontendに公開するroute、request、成功response、errorの契約所有者です。

## 責務

- `HttpOnly` Cookieと専用Redisを用いたBrowser session管理
- `matsuApi`、`toolbox`、`arcade` ごとのaccess token・refresh token・期限の分離保管
- `matsu-auth` に対するAuthorization Code + PKCE、code交換、token refresh
- 家計簿向けJSON login/registerの後方互換仲介
- `matsu-arcade-auth` に対するJSON login/register/refresh/revokeの仲介
- 3つのResource APIへの明示的route dispatchと正しいBearer JWTの選択
- Frontend向けZod schemaとOpenAPI契約の提供
- Browser requestと上流成功responseのschema検証
- timeout、通信失敗、非JSON、契約違反の安全なerrorへの正規化

## 責務に含めないもの

- 汎用catch-all proxy
- Auth Serverのlogin/register UIの提供
- ユーザー資格情報の検証やJWTへの署名
- 家計簿、Toolbox、Arcadeのdomainデータ永続化
- 他サービス向け共有Redisの提供

## routeと上流

| Frontend向けroute | 上流 | session token slot |
| --- | --- | --- |
| `/api/expenses/*`、`/api/payment-methods`、`/api/categories` | `matsu-api` | `matsuApi` |
| `/api/toolbox/*` | `matsu-toolbox-api` | `toolbox` |
| `/api/arcade/*` | `matsu-arcade-api` | `arcade` |

BFFはBrowserから受け取ったAuthorization header、Cookie、hop-by-hop headerを上流へ転送しません。routeに対応するslotのaccess tokenだけを付与し、上流URLをBrowser responseへ公開しません。

## 認証仲介

| resource | Auth Server | 現在の接続方式 | refresh |
| --- | --- | --- | --- |
| `matsuApi` | `matsu-auth` | Authorization Code + PKCE、または後方互換JSON login/register | OAuth token endpoint |
| `toolbox` | `matsu-auth` | Authorization Code + PKCE、scope `matsu-toolbox-api` | OAuth token endpoint |
| `arcade` | `matsu-arcade-auth` | BFFがJSON login/registerを仲介 | JSON `/auth/refresh` |

Arcade AuthはOAuth/OIDC Providerではありません。BFFは現行JSON契約を利用し、資格情報を保存せず、取得tokenをBrowserへ返しません。

## sessionと障害分離

Redis sessionはversion 2 schemaを持ち、3つのresource slotを独立して保存します。旧単一token sessionは `matsuApi` slotへ読み替え、Redis上の値はschema検証してから利用します。

上流 `401` では対象slotだけをrefreshして1回再試行します。refreshまたは再試行に失敗した場合は対象slotだけを削除し、全slotが空のときだけsessionとCookieを削除します。1上流のtimeout、停止、不正responseも他routeや他slotを変更しません。

全体logoutはsessionをすべて削除します。実装済みのresource単位disconnectはArcade向けで、revokeを試みた後に `arcade` slotだけを削除します。

## 技術選定

| 技術 | この構成での狙い |
| --- | --- |
| TypeScript | API契約とserver処理を同じ型systemで記述する |
| Hono | HTTP routeとmiddlewareを構成する |
| Zod | Browser request、Redis session、上流responseを実行時検証する |
| `@hono/zod-openapi` | route、schema、OpenAPIを一つの契約から構築する |
| Redis | resource別token、Browser session、短命なOAuth状態をBFF境界内で管理する |

全体のdispatch関係は[システム全体構成](../architecture/system-overview.md)、provider別の認証処理は[認証・セッション構成](../architecture/authentication.md)を参照してください。
