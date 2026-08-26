# 品質ゲート

## 目的

品質ゲートは、各リポジトリの変更がそのリポジトリの契約と実行環境に対して整合していることを、merge前に確認するための仕組みです。

`matsu` の各アプリケーションは独立したリポジトリとして開発するため、品質ゲートもリポジトリ単位で所有します。この資料は共通する考え方と現在の適用範囲を扱い、実行コマンドや環境構築手順は各リポジトリのREADMEに委ねます。

## 基本方針

- 通常実行では、変更したリポジトリのローカル品質ゲートをPull Requestを作成する前に実行する
- GitHub Actionsがあるリポジトリでは、cleanな環境で同じ品質観点を再確認する
- CI未導入のリポジトリでも、定義済みのローカル品質ゲートを省略しない
- サービス間の独立性を保ち、統合テストで開発用DBや他サービスのDBを共有しない
- 生成物をGit管理する場合は、生成元と生成物の不一致を検出する
- 実行可能な定義は各リポジトリのmanifest、テスト設定、workflowを正本とし、この資料へコマンドを重複して列挙しない

ローカル品質ゲートは、変更中の早い段階で問題を検出し、開発者が結果を確認するために使います。CIは、Pull Request上で再現可能な最小基準を強制するために使います。CIがない場合は自動判定が行われないため、実行したローカル品質ゲートと結果をレビュー時に確認します。

## 品質観点

| 観点 | 確認する内容 |
| --- | --- |
| フォーマット | 言語やリポジトリで定めた表記へ統一され、機械的な差分が混在していないこと |
| lint・静的解析 | 不正な構文、危険な記述、到達可能な型・データフロー上の問題を実行前に検出できること |
| 型チェック | TypeScriptなどの型契約が、成果物を生成せずに整合していること |
| テスト | 単体の振る舞い、公開契約、認証境界、永続化を変更に応じた粒度で確認できること |
| build | コンパイラやbundlerを通し、配布・実行に必要な成果物を生成できること |
| 生成物チェック | OpenAPIや生成型が生成元と一致し、更新漏れがないこと |

型チェックとbuildは目的が異なります。型チェックはソース全体の型整合性を短時間で確認し、buildは実際のコンパイル、bundle、成果物出力まで成立することを確認します。片方が定義されていない言語・サービスへ、別サービスの手順をそのまま持ち込みません。

## 現在の適用状況

| リポジトリ | ローカル品質ゲート | GitHub Actions |
| --- | --- | --- |
| `matsu-front` | ESLint、TypeScript型チェック、Prettier、build。BFFのOpenAPIから生成するFrontend型の整合チェックも利用可能 | `develop` / `main` 向けPull Requestで静的解析・型・フォーマットとbuildを実行。生成型整合チェックと自動テストは現在のCI対象外 |
| `matsu-bff` | ESLint、TypeScript型チェック、Prettier、OpenAPI生成物チェック、契約smoke test、build | `develop` / `main` 向けPull Requestでローカル品質ゲート一式を実行 |
| `matsu-api` | Pint、PHPStan / Larastan、PHPUnit。Git hookは変更したPHPを対象にフォーマットと静的解析を補助 | `develop` / `main` 向けPull Requestで専用MySQLを準備し、migration・seed後にPint、PHPStan、PHPUnitを実行 |
| `matsu-auth` | Docker buildでGHCの警告を含むコンパイルを確認 | 未導入。現在のCabal定義に自動テストはない |
| `matsu-toolbox-api` | ESLint、TypeScript型チェック、Prettier、単体テスト、DB統合テスト、OpenAPI生成物チェック、build | 未導入 |
| `matsu-arcade-auth` | GHC警告を有効にしたCabal buildと、単体・HTTP・DB統合を含むCabal test-suite | 未導入 |
| `matsu-arcade-api` | ESLint、TypeScript型チェック、Prettier、OpenAPI生成物チェック、単体・契約・DB統合テスト、build | 未導入 |

この表は現在実装されている範囲を示します。CI未導入の項目や現在CI対象外のチェックを、導入済みとして扱いません。

## テストデータベースの分離

DB統合テストは、開発用Composeのnamed volumeや他サービスのDBではなく、テスト対象リポジトリ専用の一時的なDBを使用します。これにより、テストの初期化・破棄が開発データや別サービスへ影響することを防ぎます。

- `matsu-api` のCIはジョブ専用のMySQLを起動し、migrationとseedを適用してからテストする
- `matsu-toolbox-api` と `matsu-arcade-api` は、`TEST_DATABASE_URL` で別途管理するテスト用PostgreSQLを指定する。通常のComposeにテスト用serviceやprofileは持たない
- `matsu-arcade-auth` のCabal test-suiteは、テスト専用に用意したPostgreSQLを指す設定で実行する。通常のCompose上の開発DBをテストDBとして共有しない

テスト用DBの起動方法、必要な環境変数、初期化方法は各リポジトリのREADMEとテスト設定を正本とします。

## 生成物の整合性

OpenAPIなどの生成物をGit管理するリポジトリでは、生成元を変更したときに生成物も同じ変更へ含めます。生成物チェックは、再生成結果とGit管理中のファイルに差分がないことを確認します。

- BFF、Toolbox API、Arcade APIは、登録済みrouteとschemaから各リポジトリのOpenAPIを生成する
- FrontendはBFFのOpenAPIからAPI型を生成する
- BFFではOpenAPI生成物チェックをCIでも実行する
- Toolbox APIとArcade APIではローカル品質ゲートとしてOpenAPI生成物を確認する。Arcade APIでは統合チェックにも含まれる
- Frontendの生成型整合チェックはローカルで利用できるが、現在のCIでは実行しない

生成元、生成先、実行方法の詳細は各リポジトリのREADMEを参照します。

## 品質ゲートを変更するとき

品質ゲートを追加・変更する場合は、そのリポジトリのmanifest、テスト設定、workflow、READMEを同じ変更で整合させます。複数サービスに共通する品質方針やCIの役割を変更する場合に、この資料も更新します。
