# Auth Server

## 位置づけ

`matsu-auth` は、`matsu-api` と `matsu-toolbox-api` を担当する簡易的なOAuth Authorization Serverです。

Browser向けlogin/register UI、JSON認証API、Authorization Code + PKCE、resource別JWT、JWKSを提供します。`matsu-arcade-auth` とは別realmであり、Arcade tokenは発行しません。

## 責務

- login・register UIとユーザー資格情報の検証
- 専用PostgreSQL上のユーザー・OAuth・refresh tokenデータ管理
- OAuth認可request、認可code、token交換
- access tokenとrefresh tokenの発行・rotation
- RS256 JWTへの署名とJWKSの公開
- `matsu-api` と `matsu-toolbox-api` のresource allowlist管理
- 選択resourceをrefresh tokenに保持し、refresh後もaudienceを維持する

## 責務に含めないもの

- BFFのBrowser application session管理
- 家計簿、Toolbox、Arcadeのdomain処理
- Arcadeユーザー、Arcade署名鍵、Arcade refresh tokenの管理
- Resource APIにおけるJWTの最終的な受け入れ判断
- 汎用的な外部IdPや完全なOpenID Connect Provider

## 複数Resource Server対応

このAuthではresource名、OAuth scope、JWT audienceを同じ文字列として扱います。

| Resource API | scope / audience | issuer |
| --- | --- | --- |
| `matsu-api` | `matsu-api` | `http://localhost:18081` |
| `matsu-toolbox-api` | `matsu-toolbox-api` | `http://localhost:18081` |

許可resourceはallowlistで検証します。JSON login/registerでaudienceを省略した場合は後方互換で `matsu-api` を選び、OAuth認可ではscopeから対象resourceを選びます。発行JWTとrefresh token行に選択したaudienceを記録するため、resource間でtokenが混ざりません。

## 概念構成

| 領域 | 役割 |
| --- | --- |
| Authentication UI | login・register formと認証error画面 |
| User Authentication | 資格情報を検証し、ユーザーを特定する |
| Authorization Server | 認可request、認可code、token交換を扱う |
| Token Service | resource別JWT署名、refresh、token rotation |
| Key Publication | JWKSとAuthorization Server metadataを公開する |
| Persistence | ユーザー、OAuth状態、audience付きrefresh tokenを専用PostgreSQLへ保存する |

## 技術選定

| 技術 | この構成での狙い |
| --- | --- |
| Haskell | 認証flowとdata typeを明示し、compile時検査を活用する |
| Servant | API typeとHTTP endpointを対応付ける |
| Lucid / `servant-lucid` | login・register UIをserver-side renderingする |
| PostgreSQL | このrealmのユーザーと認証関連データを管理する |
| Authorization Code + PKCE | Browser redirectとcode横取り耐性を両立する |
| RS256 JWT / JWKS | Authだけが秘密鍵を持ち、Resource APIは公開鍵で検証する |

## BFF・Resource APIとの境界

現在のOAuth clientは `matsu-bff` です。認可codeはBrowserを経由してBFF callbackへ返しますが、token交換とrefreshはBFFとのserver間通信です。tokenをBrowserへ直接保存させません。

`matsu-api` と `matsu-toolbox-api` は同じJWKSを利用しますが、異なるaudienceを厳密に検証します。AuthのPostgreSQLは2 APIのdomain DBやBFF Redisと共有しません。

全体のrealm関係は[システム全体構成](../architecture/system-overview.md)、flowの詳細は[認証・セッション構成](../architecture/authentication.md)を参照してください。
