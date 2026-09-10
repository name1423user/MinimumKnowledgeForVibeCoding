# 第3章　設計する

[← 目次に戻る](../README.md)

> **この章で学べること**
> - 要件と仕様の違い、抽象化・インターフェース・責務分離といった設計の基本語彙
> - 依存関係・状態管理・データ設計が、後からの変更しやすさにどう影響するか
> - 計算量やN+1問題など、パフォーマンス設計の最低限の勘所

> 🔧 **関連スキル**: [spec-first](../skills/spec-first/SKILL.md) — 要件・仕様・受け入れ条件を実装前に分離する（[skills/](../skills/README.md)参照）

## 3.1 要件と仕様

**要件**は「何を実現したいか」、**仕様**は「それをどう振る舞いとして定義するか」。「ログイン機能が欲しい」は要件、「メールとパスワードで認証し、5回失敗したらロックする」は仕様。AIに実装を頼む前に、この2つを分けて言葉にできているかを確認する。

## 3.2 抽象化

細かい実装の違いを覆い隠し、「共通して大事な部分だけ」を扱えるようにすること。「支払い方法がクレジットカードか銀行振込かは知らなくても、`pay()`さえ呼べば支払いが完了する」状態を作るのが抽象化。

## 3.3 インターフェース

「これだけ渡せば、これが返ってくる」という外部との約束事。中身の実装を差し替えても、インターフェースが変わらなければ呼び出し側は影響を受けない。

具体的に、支払い方法を追加していく場面で比較する。

```python
# 悪い例: 支払い方法が増えるたびに、呼び出し側のif文を書き換える必要がある
def pay(method, amount):
    if method == "credit_card":
        charge_credit_card(amount)
    elif method == "bank_transfer":
        request_bank_transfer(amount)
    # 新しい支払い方法（後払い、電子マネー…）が増えるたびにここへelifを継ぎ足し続ける

# 良い例: 「pay()を呼べば支払える」という約束事だけを共通化する（インターフェース）
class PaymentMethod:
    def pay(self, amount): ...  # 「amountを渡せば支払いが完了する」という約束

class CreditCardPayment(PaymentMethod):
    def pay(self, amount):
        charge_credit_card(amount)

class BankTransferPayment(PaymentMethod):
    def pay(self, amount):
        request_bank_transfer(amount)

def checkout(payment_method: PaymentMethod, amount):
    payment_method.pay(amount)  # 呼び出し側は具体的な支払い方法を知らなくていい
```

「悪い例」は動くには動くが、支払い方法が増えるたびに`pay()`関数の中身を直接書き換える必要があり、既存の分岐（`credit_card`や`bank_transfer`）を壊すリスクを毎回背負う。「良い例」は、新しい支払い方法を追加する時に**既存のクラスを1行も変更せずに**`PaymentMethod`を継承した新しいクラスを1つ足すだけで済む。これは「既存のコードを変更せずに機能を拡張できる」という設計原則（オープン・クローズドの原則）の具体例で、`checkout()`側は`PaymentMethod`という約束事（インターフェース）だけを知っていればよく、中身の実装（`CreditCardPayment`か`BankTransferPayment`か）を知らなくていい。

## 3.4 責務分離

「1つの部品は1つのことだけをする」という原則。目安は「その部品の説明に『それと』が出てきたら分割サイン」。

- 悪い例: 「ユーザーを検証**して**、DBに保存**して**、メールを送る関数」
- 良い例: `validate_user()` / `save_user()` / `send_welcome_email()`

```python
# 悪い例: 1つの関数が3つの責務を抱えている
def register_user(name, email, password):
    if not email or "@" not in email:
        raise ValueError("invalid email")
    user_id = db.insert("users", {"name": name, "email": email, "password": hash_pw(password)})
    mailer.send(email, "welcome!", "登録ありがとうございます")
    return user_id

# 良い例: 責務ごとに分割し、それぞれ単体でテストできる
def validate_user(name, email, password):
    if not email or "@" not in email:
        raise ValueError("invalid email")

def save_user(name, email, password):
    return db.insert("users", {"name": name, "email": email, "password": hash_pw(password)})

def send_welcome_email(email):
    mailer.send(email, "welcome!", "登録ありがとうございます")

def register_user(name, email, password):
    validate_user(name, email, password)
    user_id = save_user(name, email, password)
    send_welcome_email(email)
    return user_id
```

分割後は「メール送信だけ失敗した時にどうするか」のような、分割前は見えなかった仕様漏れにも気づきやすくなる。

## 3.5 依存関係

AがBを使っている時、「AはBに依存している」という。依存が多い・深いほど、Bを直した時にAが壊れるリスクが増える。「AはBの入り口（インターフェース）だけを知っていればいい」状態を目指すのが疎結合の考え方。

## 3.6 状態管理

「値が変化しうるデータ（状態）」をどこに置くかで設計の質が決まる。グローバルな場所に置くとどこからでも書き換えられて追跡不能になりやすい。必要な範囲（関数・クラス・コンポーネント）に閉じ込めるほど、影響範囲が予測できる。

## 3.7 データ設計

どんなデータを、どんな形（テーブル・構造）で持つか。後から変更しづらい部分なので、「このデータは将来どう増えうるか」を最初に考えておく価値が高い。正規化（重複を排除する設計）と非正規化（速度のためにあえて重複を許す設計）のトレードオフも知っておく。

```
-- 正規化前: 注文のたびに顧客名・住所を丸ごとコピーして持つ
orders(order_id, customer_name, customer_address, product_name, price)

-- 正規化後: 顧客情報を別テーブルに切り出し、注文からはIDで参照する
customers(customer_id, customer_name, customer_address)
orders(order_id, customer_id, product_name, price)
```

正規化前の形は、顧客が引っ越して住所が変わった時に**過去の全注文レコード**を1件ずつ更新しないと矛盾が起きる（更新漏れが起きれば、同じ顧客の注文なのに住所が食い違う）。正規化後は`customers`テーブルの1行を直すだけで済む。一方で、正規化するほど「注文一覧に顧客名を表示する」ためにテーブルを結合（JOIN）する回数が増え、参照が遅くなりやすい。アクセス頻度の高い読み取り専用の集計データ（ダッシュボードの表示用データなど）では、あえて非正規化して重複を許容し、書き込み時に整合性を保つコストを払ってでも読み取りを速くする判断も現実的にはよくある。「更新の整合性」と「読み取りの速さ」のどちらを優先するかは、そのデータが**書かれる頻度**と**読まれる頻度**のどちらが高いかで決める。

## 3.8 アーキテクチャ

システム全体をどんな部品に分け、どう繋ぐかという大枠の設計。「フロントエンドとバックエンドを分ける」「機能ごとにモジュールを分ける」といった判断がここに含まれる。最初から完璧を目指さず、「今後どこが変わりそうか」を予測して、そこだけ差し替えやすくしておくのが現実的。

## 3.9 パフォーマンス設計

「動く」の中には「速く動く」も含まれる。

- **計算量（O記法）**: データが増えた時、処理時間がどう増えるか。for文の中でfor文を回す（O(n²)）は、データ量が増えると急激に重くなる典型パターン
- **N+1問題**: 1件ずつDBに問い合わせるループを書いてしまい、本来1回で済む処理をN回実行してしまうバグ

  ```python
  # N+1問題: 注文一覧を表示するだけで、注文数+1回のクエリが発行される
  orders = db.query("SELECT * FROM orders")       # 1回目のクエリ
  for order in orders:
      customer = db.query(                        # 注文が100件あれば、ここが100回実行される
          "SELECT * FROM customers WHERE id = ?", (order.customer_id,)
      )
      print(customer.name, order.total)

  # 解決策1: JOINで一度に取得する
  rows = db.query("""
      SELECT orders.total, customers.name
      FROM orders JOIN customers ON orders.customer_id = customers.id
  """)  # クエリは常に1回

  # 解決策2: 必要なIDをまとめて集め、1回のIN句で取得する（JOINが使いにくい構成の時）
  orders = db.query("SELECT * FROM orders")
  customer_ids = [o.customer_id for o in orders]
  placeholders = ",".join(["?"] * len(customer_ids))
  customers = db.query(f"SELECT * FROM customers WHERE id IN ({placeholders})", customer_ids)
  customers_by_id = {c.id: c for c in customers}
  ```

  クエリの組み立て方自体にも注意する。件数を減らす（N+1を解消する）ことと、値を安全に渡す（[7.2節「SQL Injection」](07-security.md#72-sql-injection)のプレースホルダを使う）ことは別の問題で、両方とも満たす必要がある。上の例では`order.customer_id`はDBから読み出した値であって直接の外部入力ではないが、値をSQL文字列に直接埋め込む書き方（f-string）はSQL Injectionの温床になる書き方そのものなので、値の由来にかかわらずプレースホルダで統一しておくのが安全な習慣になる。

  注文が100件なら、N+1のコードは101回のクエリを発行するが、JOINやIN句を使えばどちらもクエリは1〜2回で済む。件数が少ないうちは体感できないが、データが増えるほど**注文件数に比例してレスポンスが遅くなる**ため、本番でユーザー数が増えてから突然「重い」と気づくパターンが多い。AIに「一覧を表示する処理を書いて」とだけ頼むと、ループの中でクエリを発行するN+1の形になりやすいので、「1回のクエリで取得して」と明示すると精度が上がる

- **キャッシュ**: 一度計算・取得した結果を保存し、再利用する。「いつ古くなるか（無効化のタイミング）」の設計が一番難しい。代表的な無効化戦略には次のようなものがある

  | 戦略 | 仕組み | 向いている場面 |
  |---|---|---|
  | TTL（有効期限） | 一定時間が経ったら自動的に古いとみなす | 多少古くても実害が小さいデータ（商品の在庫目安など） |
  | Write-through | 元データを更新すると同時にキャッシュも更新する | 常に最新であるべきデータ（ユーザーの残高表示など） |
  | イベント駆動 | 元データが変わったイベントを検知してキャッシュを消す | 更新頻度は低いが、更新後は即座に反映したいデータ |

  最も事故りやすいのは「TTLを設定し忘れて、古いキャッシュが永遠に返り続ける」パターン。キャッシュを追加する時は、保存する処理とセットで「いつ消えるか・いつ更新されるか」を必ず決めてから実装する

---

## 演習

1. 自分（またはAI）が最近書いた関数の説明を1文で書いてみる。その文に「〜して、〜して」のように「て」が2回以上出てきたら、3.4節に沿って分割案を考える。
2. 次の要件を、要件と仕様に分けて書き出す。
   > 「検索機能をつけたい」
3. 現在触っているプロジェクトで、グローバルな場所（モジュールレベル変数、シングルトンなど）に置かれている状態を1つ探し、それを必要な範囲に閉じ込めるとしたらどこに移せるか考える。
4. AIに「このAPIのレスポンスをキャッシュして」と頼む前に、キャッシュが古くなる条件（無効化のタイミング）を自分の言葉で先に書き出してから指示を出してみる。

<details>
<summary>考え方のヒント（クリックして展開）</summary>

- 演習1: 「バリデーションして**、**保存**して**、通知する」のように「て」が連続していたら、上のコード例のように関数を分割する候補
- 演習2: 「検索機能をつけたい」は要件。仕様に落とすと「商品名の部分一致で検索できる」「検索結果は関連度順に並べる」「該当0件の時は専用メッセージを出す」のように、振る舞いとして具体化されているはず
- 演習3: グローバルな設定値・キャッシュ・ログイン中ユーザー情報などが典型例。「その値を本当に使っている範囲」までスコープを狭められないか考える
- 演習4: 「元データが更新されたら」「N分経過したら」「サーバー再起動時」など、無効化条件を先に決めておくと、AIに実装させた後で「あれ、いつ古いデータが返るんだっけ」と悩まずに済む

</details>
