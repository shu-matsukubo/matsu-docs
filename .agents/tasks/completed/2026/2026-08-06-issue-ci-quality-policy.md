# Issue駆動CI委譲の品質方針を文書化する

- 状態: completed
- 優先度: normal
- 対象リポジトリ: `docs`
- 依存タスク: T01（soft dependency。方針は共有するが、本タスクの着手・完了を停止しない）

## 目的

通常のローカル品質ゲートを維持しつつ、Issue駆動実行でテストスイートをCIへ委譲できる条件と、未実行検証の記録・失敗時の継続方法を横断設計として明確にする。

## 対象範囲

- `docs/architecture/quality-gates.md` の共通方針
- Issue駆動実行におけるCI委譲条件、軽量検証、記録、CI不存在時とCI失敗時の扱い

## 作業内容

- 通常実行とIssue駆動実行の品質ゲート方針を区別して追記する
- CIへ委譲できる条件と、タスクファイルおよびPull Requestに残す検証情報を明記する
- CIが存在しない場合、CI結果を待つ場合、CI失敗後に修正する場合の扱いを明記する
- 現在のリポジトリ別CI適用状況と矛盾しないことを確認する

## 対象外

- GitHub Actions workflow、Codex skill、アプリケーションコードの変更
- 現在のCI適用状況の理想化や、未導入CIを導入済みとして記載すること
- 親ワークスペースのgitlinkまたは `modules.lock.conf` の更新

## 完了条件

- [x] 通常実行では既存のローカル品質ゲートを維持することが明記されている
- [x] Issue駆動実行でCIへ委譲できる条件、軽量検証、未実行検証の扱いが明記されている
- [x] タスクファイルとPull Requestへの記録、CI不存在時、CI待ち、CI失敗時の扱いが明記されている
- [x] 現在のCI適用状況と記述が矛盾していない
- [x] 文書向け軽量検証と自己レビューが完了している

## 実施結果

- 変更内容: 通常実行のローカル品質ゲートを維持したまま、Issue駆動実行でCIへ委譲できる前提、軽量検証、未実行結果の記録、CI不存在時とCI失敗時の扱いを `docs/architecture/quality-gates.md` に追記した。既存のリポジトリ別CI適用状況は変更していない。
- 検証結果: `git diff --check develop` 成功。`rg -n "^## " docs/architecture/quality-gates.md` で見出し構造を確認し、関連語句の検索とbase差分の目視で要件および既存表との整合を自己レビューした。docsリポジトリにはGitHub Actions、Markdown lint設定、検証用manifestがないため自動テストは未実施。文書のみの変更として軽量検証を行い、自動Markdown検証がないリスクを残す。
