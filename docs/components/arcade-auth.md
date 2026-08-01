# Arcade Auth

## 位置づけ

`matsu-arcade-auth` は、`matsu-arcade-api` 専用の独立したAuth Serverです。

email/passwordのJSON APIでregister・loginし、RS256 JWT access tokenとopaque refresh tokenを発行します。OAuth/OIDC、Browser向けlogin UI、BFF sessionは現在実装していません。

## 責務

- Arcadeユーザーの登録と資格情報検証
- access tokenとrefresh tokenの発行
- refresh tokenのrotationとrevoke
- Arcade専用RS256鍵によるJWT署名とJWKS公開
- 専用PostgreSQL上のユーザー・refresh tokenデータ管理
- raw refresh tokenを保存せず、hashで管理する

## 責務に含めないもの

- OAuth/OIDC Authorization ServerやBrowser向けlogin UI
- BFFのBrowser session管理
- Arcade profile、game、score、leaderboardの管理
- `matsu-auth` のユーザー、issuer、鍵、refresh tokenの共有
- Arcade APIのJWT受け入れ判断

## 発行token契約

| 項目 | 値 |
| --- | --- |
| Algorithm | RS256 |
| Issuer | `http://localhost:18084` |
| Audience | `matsu-arcade-api` |
| JWKS提供者 | `matsu-arcade-auth` |

## BFFとの現在の接続方式

BFFは `/auth/arcade/login` と `/auth/arcade/register` でBrowserの資格情報を受け、Arcade AuthのJSON APIへ直ちに転送します。返されたtokenはBFF Redisの `arcade` slotに保存し、Browserへ返しません。

Arcade access tokenの更新にはJSON `/auth/refresh` を使います。`/auth/arcade/disconnect` ではBFFが `/auth/revoke` を可能な範囲で呼び、他resourceを維持したままArcade slotを削除します。

## 技術とデータ境界

| 技術 | この構成での狙い |
| --- | --- |
| Haskell / Servant | 認証契約とHTTP APIを明示的なtypeで構築する |
| PostgreSQL | Arcade realm専用のユーザー・refresh tokenを管理する |
| bcrypt | password hashを保存する |
| RS256 JWT / JWKS | 専用秘密鍵で署名し、Arcade APIへ公開鍵を配布する |

専用PostgreSQLはArcade Authだけが所有し、Arcade domain DBや既存Auth DBと共有しません。自動testはhost portや開発volumeを持たない一時PostgreSQLを使い、開発用Auth DBを変更しません。

生成済みOpenAPIは持たないため、認証APIのendpoint単位の契約は実装と自動testを正本とします。BFFは認証APIの利用者であり、Browserへ公開するlogin、register、disconnectの契約はBFF側で所有します。

2つのrealmの関係は[システム全体構成](../architecture/system-overview.md)、BFF仲介flowは[認証・セッション構成](../architecture/authentication.md)、契約の管理方針は[API契約](../architecture/api-contracts.md)を参照してください。
