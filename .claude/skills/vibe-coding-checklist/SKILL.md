---
name: vibe-coding-checklist
description: 第10章の通しチェックリストを、happy-path-audit・break-it・notice-it・security-baseline・ci-readinessを横断して1回で実行する統合チェック。実装が一段落し、PR作成前や機能リリース前の最終確認に使う。
---

# vibe-coding-checklist — 通しチェックリストの実行版

対応章: [第10章 実践](../../../chapters/10-practice.md)

## 目的

[第10章](../../../chapters/10-practice.md)の「通しチェックリスト」を、個別skillを組み合わせて1回の依頼で実行する。個別に呼ぶ手間を省き、リリース前の最終確認を一括で行うためのもの。

## 使うタイミング

- 実装が一段落し、PRを作成する前
- 機能をリリースする前の最終確認
- 「一通りチェックして」とだけ依頼された時

## 手順

対象の差分・機能に対して、以下を順に実行し、結果を1つのレポートにまとめる。

1. [happy-path-audit](../happy-path-audit/SKILL.md) — ハッピーケースのみになっていないか
2. [break-it](../break-it/SKILL.md) — 見つかった問題・不足しているテストケース
3. [notice-it](../notice-it/SKILL.md) — エラーの握りつぶし・ログの欠落・責務混在
4. [security-baseline](../security-baseline/SKILL.md) — SQLi/XSS/CSRF/秘密情報/認可漏れ
5. [ci-readiness](../ci-readiness/SKILL.md) — テストが自動実行される状態になっているか

各項目を[第10章の通しチェックリスト](../../../chapters/10-practice.md#通しチェックリスト)のチェックボックスに対応させ、済/未のステータスで報告する。

## 出力フォーマット

```
## 通しチェックリスト結果: <対象機能名>

- [x] 仕様を実装前に言葉にした
- [ ] AIの実装がハッピーケースだけになっていないか確認した → 2件の異常系が未対応（詳細はhappy-path-audit参照）
- [ ] 境界値・異常系などを実際に壊してみた → break-it実施、テスト3件追加を提案
- [x] ログ・気づく仕組み → notice-it実施、問題なし
- [ ] セキュリティ → security-baseline実施、SQL Injectionの懸念1件
- [ ] CIで自動実行される状態 → ci-readiness実施、CI未設定

## 総評
未対応の項目が3件あります。優先度: セキュリティ > ハッピーケース対応 > CI導入
```

## やらないこと

- 個別skillの判定結果を上書き・簡略化しない。各skillの出力をそのまま集約する
- 「済」判定を推測で出さない。各skillを実際に実行した結果のみを反映する
