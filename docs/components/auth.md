# Auth Server

## 位置づけ

`matsu-auth`は、`matsu`のユーザー認証とトークン発行を担う、簡易的な認証プラットフォームです。

認証APIだけでなく、ブラウザ向けのログイン・ユーザー登録画面も提供します。Frontendにログインフォームを持たせず、資格情報を扱う画面と処理をAuth Serverへ集約しています。

## 責務

- ログイン・ユーザー登録画面の提供
- ユーザー資格情報の検証
- ユーザーと認証データの管理
- OAuth認可リクエストの受付
- 認可コードの発行
- アクセストークンとリフレッシュトークンの発行・更新
- RS256によるJWTアクセストークンへの署名
- JWT検証用JWKSの公開
- OAuth Authorization Serverメタデータの公開

## 責務に含めないもの

- ブラウザのアプリケーションセッション管理
- Frontend向け家計簿APIの提供
- 家計簿ドメインデータの管理
- APIにおけるJWTの最終的な受け入れ判断

## 概念構成

| 領域 | 役割 |
| --- | --- |
| Authentication UI | ログイン・登録フォームと認証エラー画面 |
| User Authentication | 資格情報を検証し、ユーザーを特定する |
| Authorization Server | 認可リクエスト、認可コード、トークン交換を扱う |
| Token Service | JWT署名、リフレッシュ、トークンローテーション |
| Key Publication | JWKSとAuthorization Serverメタデータを公開する |
| Persistence | ユーザーと認証関連データをPostgreSQLへ保存する |

## 技術選定

| 技術 | この構成での狙い |
| --- | --- |
| Haskell | 認証フローとデータ型を明示し、コンパイル時の検査を活用する |
| Servant | APIの型とHTTPエンドポイントを対応付けて構築する |
| Lucid / `servant-lucid` | ログイン・登録画面をAuth Serverでサーバーサイドレンダリングする |
| PostgreSQL | ユーザーと認証関連の永続データを管理する |
| Authorization Code + PKCE | ブラウザリダイレクトを利用しつつ、コード横取りへの耐性を持たせる |
| RS256 JWT | Auth Serverだけが秘密鍵を持ち、APIは公開鍵で検証できるようにする |
| JWKS | 署名検証用の公開鍵を標準的な形式で配布する |
| Docker Compose | Auth ServerとPostgreSQLを再現可能なローカル環境として起動する |

## OAuthクライアントとの関係

現在のローカル構成では、BFFをConfidential OAuth Clientとして扱います。

| 項目 | 現在の位置づけ |
| --- | --- |
| Client | `matsu-bff` |
| Redirect先 | BFFの認証コールバック |
| Access Tokenの対象 | `matsu-api` |
| Login UI | Auth Serverが提供 |
| Token保管 | BFFのRedis |

Auth Serverは認証成功後のアクセストークンをブラウザへ直接保存させません。認可コードをBFFへ返し、BFFがサーバー間通信でトークンへ交換します。

## 鍵の管理

開発環境では、ローカル動作用の秘密鍵とJWKSをリポジトリ内に配置しています。これは開発専用であり、本番環境の秘密情報として扱うものではありません。

本番相当の環境では、次の対応を別途設計する必要があります。

- リポジトリ外での秘密鍵管理
- 計画的な鍵ローテーション
- `kid`切り替え期間中の複数公開鍵提供
- クライアントシークレットの安全な配布と更新

全体のログイン・トークン利用フローは[認証・セッション構成](../architecture/authentication.md)を参照してください。
