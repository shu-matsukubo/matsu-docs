# 家計簿API

## 位置づけ

`matsu-api` は、家計簿domainの処理と永続データだけを管理するResource Serverです。

ネットワーク上は単体で到達可能ですが、Browserからの正式な利用経路はBFFです。`matsu-auth` が発行した `aud=matsu-api` のBearer JWTを検証し、認証済みユーザーに対するdomain処理を行います。

## 責務

- 支出、支払方法、categoryなどの家計簿データ管理
- 家計簿domainの入力検証と処理
- 専用MySQLへの永続化と検索
- `matsu-auth` が発行したJWTの検証
- JWTの `sub` を基準とした認証済みユーザーの処理

## 責務に含めないもの

- Toolbox・Arcade domainの処理
- login・register UIやユーザー資格情報の検証
- access token・refresh tokenの発行
- Browser CookieとBFF sessionの管理
- 他のBackend APIの呼び出しやDB共有

## 概念構成

| 領域 | 役割 |
| --- | --- |
| HTTP Boundary | route、入力検証、response形式 |
| Authentication | JWTを検証し、request主体を特定する |
| Application / Domain | 家計簿のuse caseと業務処理 |
| Query / Persistence | データ検索と専用MySQLへの永続化 |

## 技術選定

| 技術 | この構成での狙い |
| --- | --- |
| PHP / Laravel | routing、validation、DI、ORM、migrationを一貫して利用する |
| MySQL | 家計簿のrelational dataを専用storeへ永続化する |
| JWT | BFFからのrequestをstatelessに認証する |
| JWKS | Auth Serverの秘密鍵を共有せず、公開鍵で署名を検証する |

## JWT信頼条件

| 項目 | 値 |
| --- | --- |
| Algorithm | RS256 |
| Issuer | `http://localhost:18081` |
| Audience | `matsu-api` |
| JWKS提供者 | `matsu-auth` |

署名検証用の公開鍵は `matsu-auth` のJWKSから取得してcacheします。Toolbox向け `aud=matsu-toolbox-api` やArcade realmのtokenは受け入れません。

## BFF・DBとの境界

- BFFはBrowser sessionを `matsuApi` slotのaccess tokenへ変換する
- APIはBearer JWTを検証して家計簿domain処理を実行する
- BFFはAPI成功responseをFrontend向け契約として再検証する
- APIはFrontendのCookie、画面都合、他resourceのtokenを認識しない
- 家計簿MySQLはこのAPIだけが所有し、Auth DBや他domain DBと共有しない

家計簿APIには生成済みOpenAPIがないため、endpoint単位の契約は実装と自動testを正本とします。BFFはその契約の利用者として、Frontendへ公開する範囲を自身のschemaで定義します。

全体の通信関係は[システム全体構成](../architecture/system-overview.md)、JWTの受け渡しは[認証・セッション構成](../architecture/authentication.md)、契約の管理方針は[API契約](../architecture/api-contracts.md)を参照してください。
