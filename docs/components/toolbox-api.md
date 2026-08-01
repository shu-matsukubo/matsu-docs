# Toolbox API

## 位置づけ

`matsu-toolbox-api` は、家計簿とは独立した小さなpersonal toolを提供するResource Serverです。

`matsu-auth` が発行した `aud=matsu-toolbox-api` のBearer JWTだけを受け入れます。単体のResource APIとして直接到達可能でも、Browserからの正式な利用経路はBFFの `/api/toolbox/*` です。

## 責務

- ユーザー別noteの作成・取得・更新・削除
- ユーザー別bookmarkとtag filter
- 永続状態を持たないUnicode text検査
- 専用PostgreSQLへのToolboxデータ永続化
- `matsu-auth` が発行したToolbox向けJWTの検証
- 検証済みJWTの `sub` による所有者分離

## 責務に含めないもの

- 家計簿・Arcade domainの処理
- ユーザー認証やtoken発行
- Browser sessionとrefresh tokenの管理
- 他サービスのDB、Redis、volume、migrationの利用
- 他のBackend APIの呼び出し

## 技術選定

| 技術 | この構成での狙い |
| --- | --- |
| TypeScript / Hono | 型付きの軽量なHTTP APIを構築する |
| Zod / OpenAPI | request・responseを検証し、契約を生成する |
| `jose` | RS256 JWTとremote JWKSを検証する |
| PostgreSQL / Drizzle ORM | Toolbox専用relational dataを管理する |

## JWT信頼条件

| 項目 | 値 |
| --- | --- |
| Algorithm | RS256 |
| Issuer | `http://localhost:18081` |
| Audience | `matsu-toolbox-api` |
| OAuth scope / resource名 | `matsu-toolbox-api` |
| JWKS提供者 | `matsu-auth` |

`aud=matsu-api` やArcade realmのtokenは拒否します。所有権はemailやclient指定user IDではなく、検証済み `sub` を基準にします。

## BFF・DBとの境界

- BFFは `/api/toolbox/*` を明示的な上流routeへ対応付け、`toolbox` slotのtokenだけを送る
- APIはBearer JWTを検証し、Toolbox domain処理だけを行う
- 専用PostgreSQLをこのAPIだけが所有し、Auth DBや他domain DBと共有しない
- 自動testの実DB検証は開発DBとは別の内部network・一時PostgreSQLを使い、開発volumeを共有しない

routeとschemaから生成するOpenAPIがToolbox APIのschemaの正本です。BFFはこの契約の利用者ですが、FrontendにはBFFが所有する別の契約を公開します。

全体のrouteと所有関係は[システム全体構成](../architecture/system-overview.md)、接続・refreshは[認証・セッション構成](../architecture/authentication.md)、契約の管理方針は[API契約](../architecture/api-contracts.md)を参照してください。
