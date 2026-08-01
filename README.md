# matsu Docs

`matsu` を構成する7つのアプリケーションの役割、技術選定、認証realm、サービス間の境界を共有するための設計資料です。

このリポジトリでは、ソースコードのクラス構成や処理手順を網羅するのではなく、次の情報を扱います。

- システム全体がどのようなサービスで構成されているか
- 各サービスが何に責任を持ち、何を持たないか
- サービス間でどのように認証情報やデータを受け渡すか
- 認証データ、ドメインデータ、セッション状態を誰が所有するか
- 主要技術をどのような目的で採用しているか

## 資料構成

### 全体構成

複数のサービスをまたぐ構成や処理フローを説明します。

- [システム全体構成](docs/architecture/system-overview.md)
- [認証・セッション構成](docs/architecture/authentication.md)
- [品質ゲート](docs/architecture/quality-gates.md)
- [API契約](docs/architecture/api-contracts.md)

### 個別構成

各サービスの責務、境界、技術選定を説明します。

- [Frontend](docs/components/frontend.md)
- [BFF](docs/components/bff.md)
- [家計簿API](docs/components/api.md)
- [Auth Server](docs/components/auth.md)
- [Toolbox API](docs/components/toolbox-api.md)
- [Arcade Auth](docs/components/arcade-auth.md)
- [Arcade API](docs/components/arcade-api.md)

## 対象リポジトリ

| リポジトリ | 1行責務 |
| --- | --- |
| `matsu-front` | ブラウザ向けUIを提供し、アプリケーションAPIとしてBFFだけを利用する |
| `matsu-bff` | ブラウザセッション、resource別token、Frontend向けの明示的なAPI契約を管理する |
| `matsu-api` | 家計簿ドメインのAPIとMySQLデータを管理する |
| `matsu-auth` | 家計簿APIとToolbox API向けのOAuth認可、ユーザー認証、JWT発行を担当する |
| `matsu-toolbox-api` | メモ、ブックマーク、テキスト検査と専用PostgreSQLデータを管理する |
| `matsu-arcade-auth` | Arcade専用のJSON認証API、JWT発行、認証データを管理する |
| `matsu-arcade-api` | プレイヤー、ゲーム、スコア、ランキングと専用PostgreSQLデータを管理する |

ブラウザ向けの正式なアプリケーションAPI経路は `matsu-front` → `matsu-bff` → 各Resource APIです。`matsu-auth` と `matsu-arcade-auth` は別々の認証realmであり、ユーザーDB、署名鍵、refresh tokenを共有しません。

## 記述方針

- 「技術」「採用目的」「責務」「境界」を中心に記述する
- クラス名、関数名、細かな処理分岐などの実装詳細は各リポジトリのコードに委ねる
- セットアップ手順、ローカルURL、コマンド一覧は各リポジトリのREADMEに委ねる
- 複数サービスに関係する決定は全体構成に記述し、個別資料から参照する
- API schemaは提供サービスのOpenAPI、または実装と自動testを正本とし、この資料へ全文を転記しない
- 実装済みの現在仕様と将来案を区別し、未決定・未実装の内容を現在仕様として記述しない

技術、サービスの責務、認証フロー、データの所有者を変更した場合は、実装とあわせてこの資料も更新します。
