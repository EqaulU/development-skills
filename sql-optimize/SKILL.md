---
name: sql-optimize
description: スロークエリ改善・INDEX設計・JOIN最適化・EXPLAIN解読を行う(MySQL/PostgreSQL/SQL Server等の差分考慮)。SQL最適化・INDEX相談・実行計画レビューの依頼で使う。
---

# SQL Optimize

スロークエリの改善と INDEX 設計を診断する。

## 確認すべき情報

開始前に可能なら以下を確認:

1. **DBMS** — PostgreSQL / MySQL / SQL Server / Oracle / SQLite
2. **テーブル定義** — `\d table` / `SHOW CREATE TABLE` / `DESCRIBE`
3. **既存 INDEX** — `\di` / `SHOW INDEX FROM`
4. **実行計画** — `EXPLAIN ANALYZE` (PG) / `EXPLAIN FORMAT=JSON` (MySQL)
5. **データ量** — テーブル行数、対象データの選択性
6. **クエリの頻度** — ホットパスか、バッチで 1 日 1 回か

## 診断観点

### INDEX
- WHERE / JOIN / ORDER BY / GROUP BY のカラムに適切な INDEX があるか
- 複合 INDEX のカラム順は **等価条件 → 範囲条件 → ORDER BY 対象** の順
- カバリング INDEX で I/O 削減できないか
- 選択性の低いカラム単独の INDEX は効果が薄い
- 過剰な INDEX は書き込み性能を落とす(目安: 1 テーブルに 5-7 個まで)

### クエリ構造
- `SELECT *` → 必要カラムのみに
- サブクエリ → JOIN または EXISTS への変換可能性
- `IN (SELECT ...)` → `EXISTS` のほうが速い場合がある
- `LIKE '%xxx'`(前方一致でない) → INDEX 効かない、全文検索の検討
- `OR` 条件で複数カラム → UNION ALL に分けるほうが速い場合
- 関数適用 (`WHERE LOWER(email) = ...`) → 関数 INDEX or 計算列
- 暗黙の型変換 → INDEX 効かない原因(VARCHAR 列に数値で比較等)
- `ORDER BY ... LIMIT N` → 適切な INDEX があれば極めて速い

### N+1 とアプリ側の問題
- ループ内でクエリ → JOIN 1 発化
- ORM の `lazy load` → `eager load` 切り替え

## EXPLAIN の読み方

DBMS 別の詳細な解読チートシートと出力テンプレートは [references/explain-cheatsheet.md](references/explain-cheatsheet.md) を参照。

要点:
- **PostgreSQL**: `Seq Scan` を避け `Index Scan` / `Index Only Scan` を目指す
- **MySQL**: `type: ALL` を避け、`Extra: Using filesort/temporary` を解消する

## 原則

- **本番で計測してから判断。** 開発環境の EXPLAIN は本番と統計が違う。
- **INDEX は「タダ」ではない。** 書き込みコスト、ストレージ、メンテナンス。追加は慎重に。
- **INDEX 変更はオンライン適用を意識。** 大規模テーブルへの `CREATE INDEX` はテーブルロックの可能性。
- **DBMS 固有の機能を活用。** PostgreSQL の partial index、MySQL の covering index など、方言を意識。
- **`SELECT *` を機械的に否定しない。** カラム全部使うなら可。ただしカラム追加時の影響に注意。

## 関連スキル

- `migration-plan` — INDEX 追加・スキーマ変更を本番で安全に適用するための計画
- `perf-review` — クエリ起因以外のボトルネック(N+1 の上流、アプリ層の処理等)を一緒に診る
