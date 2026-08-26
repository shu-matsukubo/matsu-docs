# Issue駆動CI委譲の文書契約を撤去する

- 状態: completed
- タスクキー: `T2`
- 優先度: high
- 対象リポジトリ: `shu-matsukubo/matsu-docs`
- 承認識別情報: 2026-08-26の会話内で、改訂T1・T2・T3をユーザーが「T1改訂・T2・T3承認」と明示承認
- agent strategy: `worker-parent-review`
- documentation mode: `explicit-update`
- task branch: `codex/2026-08-26-remove-issue-ci-delegation`
- base branch: `develop`

agent strategyはagent種別と必須review経路を定め、人数や担当範囲を固定しない。Mainが責務境界、依存、競合、統合コストから実行時に決め、実施結果へ記録する。

実施・検証・review結果、残るrisk、documentation follow-up、agent allocation、commit、Pull Request状態、status、completed化、完了日時はbookkeepingである。目的、work、repository、completion、out-of-scope、architecture判断、dependencyの意味・種類・gate、未承認実装を変える場合だけ再計画・再承認へ戻る。

## 目的

`matsu-docs` に残るCodex Cloud / GitHub Issue orchestration専用のCI委譲契約と、その契約だけを導入した過去task記録を撤去し、Local Codex中心の品質確認方針と一致させる。

## 対象範囲

- `docs/architecture/quality-gates.md` の「Issue駆動実行でのCI委譲」節
- `.agents/tasks/completed/2026/2026-08-06-issue-ci-quality-policy.md`
- 本task fileの実施記録

## 作業内容

- Cloud / Issue Flowだけを前提とするCI委譲、実装agentのローカル実行制約、Issue上の未実施記録契約を文書から削除する。
- 退役契約だけを導入したcompleted task記録を削除する。
- 通常開発に有効な品質ゲート、既存CI一覧、Localでの検証方針を維持する。
- 退役説明・削除taskへの稼働中参照が残っていないことを横断確認する。
- Worker self review後にMain review、検証、completed化を行い、GitHub Connectorで`develop`向けdraft Pull Requestを公開する。

## 対象外

- 親`matsu-workspace`のT1差分、docs gitlink、`modules.lock.conf`の変更
- CI workflowやアプリケーションコードの変更
- 新しいIssue起点orchestration、Cloud Codex代替方式の設計
- unrelatedな文書整理・リファクタリング
- Pull Requestのmerge

## 依存関係

なし。2026-08-26にlocal / remote `develop`がともに`9afe227e13513d72dce445b509c07461e1a72419`で一致し、対象branchがremoteに存在しないことを確認した。T3は本taskとT1のPull Request mergeを`hard` / `start` gateとして待つ後続taskであり、本taskの開始依存ではない。

## 懸念事項

- Issue専用節だけを除去し、通常のCI品質ゲートやサービス別検証表を誤って削除しない。
- 退役専用task記録を削除する一方、今回のcleanup判断と検証結果は本task fileへ残す。
- 親repositoryのgitlink / lock同期はT3まで行わず、repository境界を維持する。

## 完了条件

- [x] `quality-gates.md`からCloud / GitHub Issue orchestration専用のCI委譲契約が除去されている。
- [x] 退役契約専用のcompleted task記録が削除されている。
- [x] 通常開発向けの品質ゲート、CI一覧、Local検証方針が維持されている。
- [x] 稼働中文書に削除節・削除taskへの参照が残っていない。
- [x] repositoryで定義された文書検証と`git diff --check`が成功するか、未実施理由を記録する。
- [x] Worker self reviewとMain reviewを完了する。
- [ ] `develop`向けdraft Pull Requestを公開し、URLを記録する。
- [x] 親gitlink / `modules.lock.conf`を変更せず、T3のmerge gateを維持する。

## 実施結果

- 変更内容: `docs/architecture/quality-gates.md`から「Issue駆動実行でのCI委譲」節だけを削除し、同契約だけを導入した`.agents/tasks/completed/2026/2026-08-06-issue-ci-quality-policy.md`を削除した。通常のLocal品質方針、品質観点、リポジトリ別CI適用表、DB分離、生成物整合方針は維持した。
- ローカル検証: 稼働中文書の退役参照検索0件。全Markdown 12ファイルの相対link確認成功。`quality-gates.md`の見出し構造確認成功。`git diff --check develop`と`git diff --check develop...HEAD`成功。base差分と変更範囲を目視確認した。
- CI: このrepositoryにはGitHub Actions、Markdown lint設定、検証manifestが存在しないため自動CIは未実施。文書変更向けの上記軽量検証で確認した。
- documentation follow-up: なし
- agent allocation・実行結果: Main 1名、Worker 1名。対象2ファイルが同一文書責務へ強く結合するためWorker 1名へまとめて割り当てた。Herschelが指定2ファイルを実装し、参照・link・見出し・diff checkを実行、self review findingなし。Mainが統合差分、通常品質方針の保持、repository境界を確認し、actionable findingなし。
- commit: `053f8ec` task定義、`f286149` 文書cleanup実装。completed記録commitは本更新で作成する。
- Pull Request: 公開準備中
- 完了日時: 2026-08-26（draft Pull Request URL記録待ち）
