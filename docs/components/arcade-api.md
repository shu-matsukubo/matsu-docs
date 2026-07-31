# Arcade API

## 位置づけ

`matsu-arcade-api` は、Arcade domainを管理するResource Serverです。

`matsu-arcade-auth` が発行した `aud=matsu-arcade-api` のBearer JWTだけを信頼します。単体のResource APIとして直接到達可能でも、Browserからの正式な利用経路はBFFの `/api/arcade/*` です。

## 責務

- player profileの管理
- game catalogの提供
- user別scoreの登録・取得・削除
- game別leaderboardの計算
- 専用PostgreSQLへのArcade domainデータ永続化
- Arcade Authが発行したJWTの検証
- 検証済みJWTの `sub` による所有者分離

## 責務に含めないもの

- game本体やBrowser UIの実装
- ユーザー資格情報の検証とtoken発行
- Browser sessionやrefresh tokenの管理
- `matsu-auth` のtoken受け入れ
- 他のBackend APIの呼び出しやDB共有

## 技術選定

| 技術 | この構成での狙い |
| --- | --- |
| TypeScript / Hono | 型付きのHTTP Resource APIを構築する |
| Zod / OpenAPI | request・responseの実行時検証と契約生成を行う |
| `jose` | RS256 JWTとArcade Auth JWKSを検証する |
| PostgreSQL / Drizzle ORM | profile、game、scoreを専用storeで管理する |

## JWT信頼条件

| 項目 | 値 |
| --- | --- |
| Algorithm | RS256 |
| Issuer | `http://localhost:18084` |
| Audience | `matsu-arcade-api` |
| JWKS提供者 | `matsu-arcade-auth` |

`matsu-auth` が発行する `iss=http://localhost:18081` のtoken、家計簿・Toolbox向けaudienceのtokenは拒否します。所有権とleaderboard上の主体は検証済み `sub` を基準にし、emailをownership keyにしません。

## BFF・DBとの境界

- BFFは `/api/arcade/*` を明示的な上流routeへ対応付け、`arcade` slotのtokenだけを送る
- APIはBearer JWTを検証し、Arcade domain処理だけを行う
- 専用PostgreSQLをこのAPIだけが所有し、Arcade Auth DBや他domain DBと共有しない
- APIはArcade AuthからJWKSを取得する以外、AuthのDBやrefresh tokenを認識しない

全体のrouteと所有関係は[システム全体構成](../architecture/system-overview.md)、realmとtokenの受け渡しは[認証・セッション構成](../architecture/authentication.md)を参照してください。
