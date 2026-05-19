---
name: sql-optimize
description: スロークエリ改善・INDEX設計・JOIN最適化・EXPLAIN解読を行う(MySQL/PostgreSQL/SQL Server等の差分考慮)。SQL最適化・INDEX相談・実行計画レビューの依頼で使う。
---

# SQL Optimize

スロークエリの改善とINDEX設計を診断する。

## 確認すべき情報

開始前に、可能であれば以下を確認する:

1. **DBMS** — PostgreSQL / MySQL / SQL Server / Oracle / SQLite(方言が異なる)
2. **テーブル定義** — `\d table` / `SHOW CREATE TABLE` / `DESCRIBE`
3. **既存INDEX** — `\di` / `SHOW INDEX FROM`
4. **実行計画** — `EXPLAIN ANALYZE` (PG) / `EXPLAIN FORMAT=JSON` (MySQL)
5. **データ量** — テーブル行数、対象データの選択性
6. **クエリの頻度** — ホットパスか、バッチで1日1回か

## 診断観点

### INDEX
- WHERE / JOIN / ORDER BY / GROUP BY のカラムに適切なINDEXがあるか
- 複合INDEXのカラム順は **等価条件 → 範囲条件 → ORDER BY対象** の順
- カバリングINDEX(必要カラムをINDEXに含める)で I/O削減できないか
- 選択性の低いカラム(boolean等)単独のINDEXは効果が薄い
- 過剰なINDEXは書き込み性能を落とす(目安: 1テーブルに5-7個まで)

### クエリ構造
- `SELECT *` → 必要カラムのみに
- サブクエリ → JOIN または EXISTS への変換可能性
- `IN (SELECT ...)` → `EXISTS` のほうが速い場合がある
- `LIKE '%xxx'` (前方一致でない) → INDEX効かない、全文検索の検討
- `OR` 条件で複数カラム → UNION ALL に分けるほうが速い場合
- 関数適用 (`WHERE LOWER(email) = ...`) → 関数INDEX or 計算列
- 暗黙の型変換 → INDEX効かない原因(VARCHAR列に数値で比較等)
- `ORDER BY ... LIMIT N` → 適切なINDEXがあれば極めて速い

### N+1とアプリ側の問題
- ループ内でクエリ → JOIN 1発化
- ORM の `lazy load` → `eager load` 切り替え

### EXPLAIN の読み方

**PostgreSQL**:
- `Seq Scan` (全表走査) — INDEXが無いか効いていない
- `Index Scan` / `Index Only Scan` — 望ましい
- `Bitmap Heap Scan` — 中程度の選択性。多くの行をINDEXで絞る場合
- `Nested Loop` × 大きなテーブル — 危険。Hash JoinやMerge Joinが望ましいことも
- `rows=` が実態と乖離 → 統計情報古い、`ANALYZE`実行を推奨

**MySQL**:
- `type: ALL` — 全表走査(避けたい)
- `type: range` / `ref` / `eq_ref` / `const` — INDEX利用
- `Extra: Using filesort` / `Using temporary` — 重い処理
- `Extra: Using index` — カバリングINDEXが効いている(良い)

## 出力フォーマット

```markdown
## SQL最適化提案

### 現状の問題
- `SELECT * FROM orders WHERE user_id = ? ORDER BY created_at DESC LIMIT 20`
- 実行計画: `Seq Scan on orders (cost=... rows=500000)`
- 推定: 50万行全表走査 → ~800ms

### 改善案 1: 複合INDEX追加(推奨)
\`\`\`sql
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at DESC);
\`\`\`
- 期待効果: 800ms → ~5ms
- カラム順の理由: user_id は等価条件、created_at は ORDER BY なのでこの順
- 副作用: orders への INSERT/UPDATE が ~10% 遅くなる(許容範囲のはず)

### 改善案 2: SELECT * の削減
\`\`\`sql
SELECT id, total, status, created_at FROM orders WHERE ...
\`\`\`
- 上記INDEXに `total, status` を含めればカバリングINDEXになる

### 確認ポイント
- 本番のデータ分布で本当に user_id の選択性が高いか
- 既存INDEXとの重複がないか
- INDEX作成時のロック影響(`CREATE INDEX CONCURRENTLY` 推奨 / PG)
```

## 原則

- **本番で計測してから判断。** 開発環境のEXPLAINは本番と統計が違う。
- **INDEXは「タダ」ではない。** 書き込みコスト、ストレージ、メンテナンス。追加は慎重に。
- **INDEX変更はオンライン適用を意識。** 大規模テーブルへの`CREATE INDEX`はテーブルロックの可能性。
- **DBMS固有の機能を活用。** PostgreSQLのpartial index、MySQLのcovering indexなど、方言を意識。
- **`SELECT *` を機械的に否定しない。** カラム全部使うなら可。ただしカラム追加時の影響に注意。

## 関連スキル

- `migration-plan` — INDEX 追加・スキーマ変更を本番で安全に適用するための計画
- `perf-review` — クエリ起因以外のボトルネック(N+1の上流、アプリ層の処理等)を一緒に診る
