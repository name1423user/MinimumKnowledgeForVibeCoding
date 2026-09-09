# 第10章　実践

[← 目次に戻る](../README.md)

> **この章で学べること**
> - これまでの全章の知識を、1つの小さなアプリで実際に通しで使う感覚
> - 「知識→実装→検証」のループを自分の手で1周させる具体的な手順

知識を頭に入れただけでは身につかない。小さなアプリを題材に、一連の流れを実際に回す。

1. **AIに小さなアプリを作らせる** — 仕様はあえて簡潔に伝える（[9.5節](09-choosing-ai-tools.md#95-選び方の指針)を参考にツールを選ぶ）
2. **わざと仕様漏れを入れる** — 「空欄のまま送信したら」のようなケースを最初は伝えない
3. **AIに実装させる** — ハッピーケースだけの実装が出てくることを確認する（[1.4節](01-ai-writes-code.md#14-aiがハッピーケースしか考えない理由)）
4. **自分でレビューする** — [第2〜3章](02-reading-code.md)の知識を使って、違和感に気づけるか試す
5. **壊してみる** — [第5章](05-breaking-it.md)の観点で、実際に境界値・異常系を試す
6. **修正する** — 見つかった問題をAIに直させる。この時、精度の高い指示（[第4章](04-supervising-ai.md)）が効いてくる
7. **壊し方をテスト化する** — 見つけたバグをテストコードとして残す（[5.11節](05-breaking-it.md#511-壊し方をテストコードに変換する)）
8. **CIに載せる** — テストを毎回自動実行する仕組みを作る（[第8章](08-testing-and-cicd.md)）
9. **気づく仕組みを入れる** — ログや通知を仕込み、次に壊れた時に自分で気づけるようにする（[第6章](06-noticing.md)）
10. **セキュリティを見直す** — 外部入力の扱いに穴がないか、[第7章](07-security.md)のチェックリストで確認する
11. **最後に設計を改善する** — 一通り動いた後で、責務分離やデータ設計（[第3章](03-design.md)）に無理がないか見直す

このループを一度でも自分の手で回せれば、以降のバイブコーディングの質が確実に変わる。

## 具体例: メモAPIを作ってみる

抽象的な手順だけだとイメージしづらいので、「メモを保存・一覧表示するAPI」を題材に、実際のコードでこのループを1周する。

### 1〜3. 仕様を簡潔に伝えて実装させる

AIに次のように依頼したとする。

> 「POST /notes でテキストを受け取って保存、GET /notes で保存したメモの一覧を返すAPIを作って」

わざと「空文字だったら」「同時に大量に送られたら」には触れていない。返ってきた実装は、典型的にこうなる（[1.4節](01-ai-writes-code.md#14-aiがハッピーケースしか考えない理由)通りのハッピーケース実装）。

```python
notes = []

@app.route("/notes", methods=["POST"])
def create_note():
    text = request.json["text"]
    notes.append(text)
    return {"ok": True}

@app.route("/notes")
def list_notes():
    html = "".join(f"<li>{n}</li>" for n in notes)
    return f"<ul>{html}</ul>"
```

### 4. レビューする

[第2〜3章](02-reading-code.md)の知識で読むと、違和感が見つかる。

- `request.json["text"]` — `text`キーが無ければ`KeyError`で落ちる（[2.4節](02-reading-code.md#24-エラー処理)のエラー処理が無い）
- メモの一覧をそのままHTMLに埋め込んでいる（[7.3節](07-security.md#73-xssクロスサイトスクリプティング)のXSSの典型パターン）
- `notes`がグローバルなリストで、責務も検証・保存・表示が1つの関数に混ざっている（[3.4節](03-design.md#34-責務分離)・[3.6節](03-design.md#36-状態管理)）

### 5. 壊してみる

[第5章](05-breaking-it.md)の観点で実際に叩く。

- `{"text": ""}` を送る → 空メモがそのまま保存される（空データ）
- `{"text": "<script>alert(1)</script>"}` を送り、一覧を表示する → スクリプトが実行される（異常系・セキュリティ）
- `text`キー無しで送る → 500エラーで落ちる（異常系）

### 6〜7. 修正し、テスト化する

見つかったバグを直し、そのまま[5.11節](05-breaking-it.md#511-壊し方をテストコードに変換する)の要領でテストにする。

```python
from markupsafe import escape

def validate_note(payload):
    text = payload.get("text", "")
    if not text.strip():
        raise ValueError("text is required")
    return text

def save_note(text):
    notes.append(text)

@app.route("/notes", methods=["POST"])
def create_note():
    text = validate_note(request.json or {})
    save_note(text)
    return {"ok": True}

@app.route("/notes")
def list_notes():
    html = "".join(f"<li>{escape(n)}</li>" for n in notes)
    return f"<ul>{html}</ul>"
```

```python
import pytest

def test_empty_text_is_rejected():
    with pytest.raises(ValueError):
        validate_note({"text": ""})

def test_missing_text_key_is_rejected():
    with pytest.raises(ValueError):
        validate_note({})

def test_html_in_note_is_escaped(client):
    client.post("/notes", json={"text": "<script>alert(1)</script>"})
    res = client.get("/notes")
    assert "<script>" not in res.text
```

### 8〜11. CI・ログ・セキュリティ・設計の仕上げ

- **CI**（[第8章](08-testing-and-cicd.md)）: 上のテストをpushのたびに自動実行する設定を追加する
- **ログ**（[第6章](06-noticing.md)）: `validate_note`が`ValueError`を投げた時、どんな入力で失敗したかを記録する
- **セキュリティ**（[第7章](07-security.md)）: `escape()`でXSSを防いだ他に、メモの長さに上限を設けて巨大データ攻撃も防ぐ
- **設計**（[第3章](03-design.md)）: `validate_note` / `save_note` / 表示処理が分離済みなので、次に「メモを編集する」機能を足す時も影響範囲が予測しやすい

たった1つのエンドポイントでも、11ステップ全部を通すと本書のほぼ全章が顔を出す。これが「知識→実装→検証」のループの正体だ。

## 通しチェックリスト

実践課題を進める際、次のチェックリストを印刷代わりに使うといい。

- [ ] 仕様（何を満たせば正しいか）を、実装前に言葉で書いた
- [ ] AIの実装がハッピーケースだけになっていないか確認した
- [ ] 境界値・異常系・空データ・同時実行のいずれかで実際に壊してみた
- [ ] 見つけたバグをテストコードとして残した
- [ ] テストがpushのたびに自動実行される状態になっている
- [ ] 失敗した時にログか通知で気づける状態になっている
- [ ] 外部入力をそのままSQL・HTML・シェルコマンドに埋め込んでいないか確認した
- [ ] 秘密情報がコードに直書きされていない
- [ ] 1つの関数・部品が1つの責務だけを持っているか見直した

---

## 演習

1. 上のチェックリストを使って、自分が最近AIと一緒に作った機能（小さなものでよい）を1つ、最初から最後まで通しで採点してみる。チェックが付かなかった項目を1つ選び、実際に手を動かして埋める。
2. このループを2回目以降回す時、1回目より速く終わった工程・変わらず時間がかかる工程を記録する。時間がかかる工程があれば、それがどの章の知識不足に由来するか特定する。

<details>
<summary>考え方のヒント（クリックして展開）</summary>

- 演習1: 上の「メモAPI」の例のように、チェックが付かなかった項目はたいてい「壊してみる」か「テスト化する」のどちらか。まずその2工程だけでも実施すると、採点が大きく改善する
- 演習2: 「壊す」工程に時間がかかるなら[第5章](05-breaking-it.md)、「なぜ壊れたか追えない」なら[第6章](06-noticing.md)、というように詰まる工程と対応する章はほぼ1対1になっている

</details>
