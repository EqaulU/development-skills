# EXPLAIN 解読チートシート

`sql-optimize` skill から参照される、DBMS 別の実行計画読み方リファレンス。

## PostgreSQL

| ノード | 意味 |
| :--- | :--- |
| `Seq Scan` | 全表走査 — INDEX が無いか効いていない |
| `Index Scan` | INDEX 利用 — 望ましい |
| `Index Only Scan` | カバリング INDEX — さらに望ましい(テーブル本体に行かない) |
| `Bitmap Heap Scan` | 中程度の選択性。多くの行を INDEX で絞る場合 |
| `Nested Loop` × 大きなテーブル | 危険。Hash Join や Merge Join が望ましいことも |

その他:
- `rows=` が実態と乖離 → 統計情報古い、`ANALYZE` 実行を推奨
- `Buffers:` を見れば実際の I/O 量が分かる(`EXPLAIN (ANALYZE, BUFFERS)`)

## MySQL

| `type` | 意味 |
| :--- | :--- |
| `ALL` | 全表走査(避けたい) |
| `range` | 範囲スキャン — INDEX 利用 |
| `ref` | INDEX 利用(非ユニーク) |
| `eq_ref` | INDEX 利用(ユニーク) |
| `const` | 1 行確定 — 最速 |

`Extra` フィールド:
- `Using filesort` — 重い処理(ソートに INDEX が使えていない)
- `Using temporary` — 一時テーブル使用、重い
- `Using index` — カバリング INDEX が効いている(良い)
- `Using where` — フィルタが行レベルで適用されている

## 共通の見方

- まず**全表走査**(`Seq Scan` / `type: ALL`)を疑う
- 次に**JOIN の戦略**(Nested Loop / Hash / Merge)が適切か
- 最後に**ソート/グループ化**(filesort/temporary)が必要なら INDEX で回避できないか

## 出力テンプレート例

```markdown
## SQL 最適化提案

### 現状の問題
- `SELECT * FROM orders WHERE user_id = ? ORDER BY created_at DESC LIMIT 20`
- 実行計画: `Seq Scan on orders (cost=... rows=500000)`
- 推定: 50 万行全表走査 → ~800ms

### 改善案 1: 複合 INDEX 追加(推奨)
\`\`\`sql
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at DESC);
\`\`\`
- 期待効果: 800ms → ~5ms
- カラム順の理由: user_id は等価条件、created_at は ORDER BY なのでこの順
- 副作用: orders への INSERT/UPDATE が ~10% 遅くなる

### 改善案 2: SELECT * の削減
\`\`\`sql
SELECT id, total, status, created_at FROM orders WHERE ...
\`\`\`
- 上記 INDEX に `total, status` を含めればカバリング INDEX になる

### 確認ポイント
- 本番のデータ分布で本当に user_id の選択性が高いか
- 既存 INDEX との重複がないか
- INDEX 作成時のロック影響(`CREATE INDEX CONCURRENTLY` 推奨 / PG)
```
