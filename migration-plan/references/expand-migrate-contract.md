# Expand-Migrate-Contract パターン 詳細

`migration-plan` skill から参照される、ゼロダウンタイム DB スキーマ変更パターン。

**例: `users.fullname` を `first_name` + `last_name` に分割したい**

## Phase 1: Expand(追加のみ、後方互換)

```sql
ALTER TABLE users ADD COLUMN first_name VARCHAR(255);
ALTER TABLE users ADD COLUMN last_name VARCHAR(255);
```

- アプリ側: 書き込み時に新旧両方に書く(Dual Write)
- 旧コードは引き続き `fullname` を読める

## Phase 2: Backfill(既存データ移行)

```sql
-- 既存行を埋める(バッチで段階的に)
UPDATE users
SET first_name = SPLIT_PART(fullname, ' ', 1),
    last_name  = SPLIT_PART(fullname, ' ', 2)
WHERE first_name IS NULL
LIMIT 10000;  -- ループでバッチ実行
```

- ロック時間最小化のため小ロットで
- 進捗ログを残す

## Phase 3: Migrate(読み取り側を新カラムに切替)

- アプリの SELECT を新カラムに切替えてデプロイ
- 引き続き Dual Write は維持(ロールバック余地のため)

## Phase 4: Contract(旧カラム削除)

- Dual Write を停止(新カラムのみに書く)
- 十分な観測期間(数日〜数週)を置く
- 最後に旧カラム削除

```sql
ALTER TABLE users DROP COLUMN fullname;
```

## 出力テンプレート(マイグレーション計画書)

```markdown
## マイグレーション計画: <変更内容>

### 概要
何を、なぜ変更するか(1-2文)

### 変更の影響評価
- 種別: Breaking(Expand-Migrate-Contract 必要)
- 対象テーブル: `users`(行数: 約500万)
- アクセス頻度: 1000 req/s(読み)、50 req/s(書き)
- ダウンタイム許容: 0秒

### Phase 1: Expand
- マイグレーション: ...
- アプリ変更: Dual Write 実装
- デプロイ: ...
- 検証: ...
- ロールバック: マイグレーションの down で戻せる(NULLABLE なので無害)

### Phase 2: Backfill
- スクリプト: `scripts/backfill_user_names.sql`
- 実行方法: 10000行ずつバッチ、ピーク外に実行、所要時間 ~2h 見込み
- 監視: スループット、レプリカ遅延、ロック待ち
- 中断時の再開可能性: WHERE 条件で未処理行のみ対象

### Phase 3: Migrate
- アプリの SELECT 切替(feature flag でカナリア展開推奨)
- 観測: 新カラムベースの読み取りでエラーが出ないか
- ロールバック: feature flag OFF で即時切り戻し

### Phase 4: Contract
- 前提: 7 日間問題なし、新コードのみ稼働
- マイグレーション: DROP COLUMN
- 注意: 一度実行するとロールバック不可。十分な事前確認

### 各 Phase の Go/No-Go 判定基準
- エラー率変化なし
- レイテンシ変化なし
- レプリカ遅延が閾値以下
- 関係者の合意

### リハーサル計画
- ステージング環境で本番相当データ量で実施
- バックアップからのリストア訓練
```
