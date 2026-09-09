# skills — 本書を実行可能にする指示集

[← 目次に戻る](../README.md)

このフォルダには、本書の各章の考え方を、実際のコードに対して実行できる形にした指示ファイル（`SKILL.md`）が入っている。読んで終わりにせず、[第10章「実践」](../chapters/10-practice.md)の通しチェックリストを自分の手を動かして回すための道具、という位置づけ。

## 他のAIツールでも使える

各`SKILL.md`はプレーンなMarkdownの指示文で、特定のツールに縛られるコードは含んでいない。

- **Claude Code**: プロジェクトの`.claude/skills/`に置くと、`description`の内容に応じて自動的に候補に挙がる。このリポジトリでは意図的に`.claude/skills/`ではなく`skills/`に置いているため、Claude Codeで自動読み込みさせたい場合は、使いたいものを`.claude/skills/`にコピーするか、シンボリックリンクを作る必要がある
- **その他のAIツール（ChatGPT、Cursorなど）**: 自動発火の仕組みはないが、`SKILL.md`の本文をシステムプロンプト・カスタム指示・都度のプロンプトとして渡せば、同じ内容で使える

## 一覧

| skill | 対応章 | 役割 |
|---|---|---|
| [spec-first](spec-first/SKILL.md) | [3章](../chapters/03-design.md)・[4章](../chapters/04-supervising-ai.md) | 作る前 — 要件/仕様/受け入れ条件の分離、仕様変更時の再レビュー |
| [happy-path-audit](happy-path-audit/SKILL.md) | [1章](../chapters/01-ai-writes-code.md)・[5章](../chapters/05-breaking-it.md) | 作った直後 — ハッピーケース検出＋過剰実装（未依頼の動作）検出 |
| [break-it](break-it/SKILL.md) | [5章](../chapters/05-breaking-it.md) | テスト — 壊し方を3系統に分類し、結果を3種に判定（バグ/期待通り/仕様不明） |
| [notice-it](notice-it/SKILL.md) | [2章](../chapters/02-reading-code.md)・[3章](../chapters/03-design.md)・[6章](../chapters/06-noticing.md) | 本番 — ログ→検知→通知→調査の観測可能性、アラート疲れを避ける通知基準 |
| [security-baseline](security-baseline/SKILL.md) | [7章](../chapters/07-security.md) | 攻撃者視点 — 前提確認（出所→信頼境界→使用箇所）＋秘密情報の漏洩経路 |
| [ci-readiness](ci-readiness/SKILL.md) | [8章](../chapters/08-testing-and-cicd.md) | 継続運用 — CIの信頼性、CI/本番環境差の確認 |
| [ai-tool-picker](ai-tool-picker/SKILL.md) | [9章](../chapters/09-choosing-ai-tools.md) | タスクに応じたAIツール形態の助言 |
| [vibe-coding-checklist](vibe-coding-checklist/SKILL.md) | [10章](../chapters/10-practice.md) | 上記をフィードバックループとして実行する統合版（停止条件つき） |

## 全体の流れ

`vibe-coding-checklist`のMermaid図がパイプライン全体を表している。要点は、一方向のチェックリストではなく、バグは`happy-path-audit`へ、仕様不明は`spec-first`へ戻る**フィードバックループ**になっていることと、CRITICALな問題を見つけたら即座に停止して人間の判断に戻ること。詳細は[vibe-coding-checklist/SKILL.md](vibe-coding-checklist/SKILL.md#停止条件critical)を参照。

## 設計方針（共通）

- **指摘・提案までで、無断修正はしない**（[4.6節「AIの回答を鵜呑みにしない」](../chapters/04-supervising-ai.md#46-aiの回答を鵜呑みにしない)の姿勢と一致させる）
- 判定には必ず本書の該当節へのリンクを付ける
- 出力フォーマットを表形式に統一する
