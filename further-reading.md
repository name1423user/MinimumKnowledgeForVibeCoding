# 参考資料・次に読むべきもの

[← 目次に戻る](README.md)

本書は「最低限」に絞っている。物足りなくなったら、次はこのあたりを深掘りするといい。

## コードの読み書き・設計

- **『リーダブルコード』**（Dustin Boswell, Trevor Foucher）— [第2章](chapters/02-reading-code.md)・[第3章](chapters/03-design.md)で扱った「読みやすさ」「責務分離」を、コードレベルの具体例で学べる定番書
- 使用している言語・フレームワークの**公式ドキュメント** — [付録](appendix.md)で触れた通り、まず公式を優先する。AIが提示する情報より新しく正確なことが多い

## テスト

- **『テスト駆動開発』**（Kent Beck）— [第5章](chapters/05-breaking-it.md)・[第8章](chapters/08-testing-and-cicd.md)のテストの考え方を、実際に手を動かしながら体系的に学べる
- 使用しているテストフレームワーク（pytest、Jest など）の公式ドキュメントの「テストピラミッド」「モック」に関する章
- **Hypothesis（Python）** (<https://hypothesis.readthedocs.io/>) — [5.3節](chapters/05-breaking-it.md#53-手で1つずつ壊す限界-プロパティベーステストファジング)で触れたプロパティベーステストの代表的なライブラリ。JavaScriptなら`fast-check`が近い立ち位置

## セキュリティ

- **OWASP Top 10** (<https://owasp.org/www-project-top-ten/>) — [第7章](chapters/07-security.md)で扱った脆弱性の、業界標準の分類と最新動向
- **OWASP Top 10 for Large Language Model Applications** (<https://owasp.org/www-project-top-10-for-large-language-model-applications/>) — [9.3.1節](chapters/09-choosing-ai-tools.md#931-エージェント型特有のリスク-プロンプトインジェクション)で触れたプロンプトインジェクションなど、AIエージェント特有のリスクの分類
- **IPA（情報処理推進機構）の情報セキュリティ関連資料** (<https://www.ipa.go.jp/security/>) — 日本語で読める、実例に基づいたセキュリティ情報

## CI/CD

- 利用しているCIサービス（GitHub Actions、GitLab CI など）の公式ドキュメントの「はじめに」— [第8章](chapters/08-testing-and-cicd.md)で扱った概念を、実際の設定ファイルに落とし込む際に必須

## AIとの協働

- 利用しているAIツールの公式ドキュメント・リリースノート — [第9章](chapters/09-choosing-ai-tools.md)で挙げた形態（チャット型・IDE統合型・CLI/エージェント型）は進化が速く、できることが数ヶ月単位で変わる。本書の分類より、常に一次情報を優先する

## 見つけたら追加してほしいもの

このリストは完成形ではない。役に立った資料があれば、この本の運用者に共有・追記してもらう形で育てていくのがいい。
