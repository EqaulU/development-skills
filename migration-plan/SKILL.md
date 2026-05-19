---
name: migration-plan
description: DBスキーマ変更・データ移行の計画を作成する(Expand-Migrate-Contractパターン、オンラインDDL、ロールバック手順)。マイグレーション・スキーマ変更・ゼロダウンタイムの依頼で使う。
---

# Migration Plan

DBスキーマ変更とデータ移行の計画を立てる。

## 入力情報

開始時に確認:

- **DBMS** — PostgreSQL / MySQL / Oracle / SQL Server / SQLite
- **テーブル規模** — 行数、サイズ(MB/GB/TB)
- **アクセスパターン** — 読み書き頻度、ピーク時間帯
- **ダウンタイム許容** — 0秒 / 数分 / 数十分 / メンテナンス窓
- **デプロイ方式** — Blue/Green / Rolling / In-place

## マイグレーション分類

| 種別 | リスク | 例 |
| :--- | :--- | :--- |
| **後方互換(Additive)** | 低 | カラム追加(NULLABLE)、INDEX追加、テーブル追加 |
| **両方向互換(Compatible)** | 中 | カラム改名(エイリアス併用)、NOT NULL化(既存データ充足済み) |
| **破壊的(Breaking)** | 高 | カラム削除、型変更、NOT NULL化(NULL残存)、テーブル削除 |

破壊的変更は **Expand-Migrate-Contract** パターンで段階分割する。

## Expand-Migrate-Contract パターン

**例: `users.fullname` を `first_name` + `last_name` に分割したい**

### Phase 1: Expand(追加のみ、後方互換)
```sql
ALTER TABLE users ADD COLUMN first_name VARCHAR(255);
ALTER TABLE users ADD COLUMN last_name VARCHAR(255);
```
- アプリ側: 書き込み時に新旧両方に書く(Dual Write)
- 旧コードは引き続き `fullname` を読める

### Phase 2: Backfill(既存データ移行)
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

### Phase 3: Migrate(読み取り側を新カラムに切替)
- アプリのSELECTを新カラムに切替えてデプロイ
- 引き続き Dual Write は維持(ロールバック余地のため)

### Phase 4: Contract(旧カラム削除)
- Dual Write を停止(新カラムのみに書く)
- 十分な観測期間(数日〜数週)を置く
- 最後に旧カラム削除
```sql
ALTER TABLE users DROP COLUMN fullname;
```

## オンラインDDL / ロック影響

### PostgreSQL
- `CREATE INDEX CONCURRENTLY` — 読み書きをブロックしない
- `ALTER TABLE ADD COLUMN ... DEFAULT ...` — PG11以降は高速(メタデータのみ)
- `ALTER TABLE ALTER COLUMN ... TYPE ...` — テーブル全体のリライトが発生する場合あり、要注意
- `pg_repack` で長時間ロックを回避

### MySQL
- `ALGORITHM=INPLACE, LOCK=NONE` を明示
- `ALTER TABLE` のオンライン可否は変更内容次第。マニュアルで要確認
- 大規模変更は `pt-online-schema-change` / `gh-ost` を使用

### 共通の注意
- 5GB以上のテーブルへのDDLは特に慎重に
- INDEX作成はピーク時間外を選ぶ
- 長時間トランザクションの後ろにDDLが詰まると致命的

## 出力フォーマット

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
- ロールバック: マイグレーションのdownで戻せる(NULLABLEなので無害)

### Phase 2: Backfill
- スクリプト: `scripts/backfill_user_names.sql`
- 実行方法: 10000行ずつバッチ、ピーク外に実行、所要時間 ~2h見込み
- 監視: スループット、レプリカ遅延、ロック待ち
- 中断時の再開可能性: WHERE条件で未処理行のみ対象

### Phase 3: Migrate
- アプリのSELECT切替(feature flagでカナリア展開推奨)
- 観測: 新カラムベースの読み取りでエラーが出ないか
- ロールバック: feature flag OFFで即時切り戻し

### Phase 4: Contract
- 前提: 7日間問題なし、新コードのみ稼働
- マイグレーション: DROP COLUMN
- 注意: 一度実行するとロールバック不可。十分な事前確認

### 各Phase の Go/No-Go 判定基準
- エラー率変化なし
- レイテンシ変化なし
- レプリカ遅延が閾値以下
- 関係者の合意

### リハーサル計画
- ステージング環境で本番相当データ量で実施
- バックアップからのリストア訓練
```

## 原則

- **本番DBの変更はリハーサルが必須。** ステージングで本番相当のデータ量でテストする。
- **後方互換を最大限維持する設計。** 「アプリとDBが一瞬不一致になる時間帯」が出ないように。
- **大規模UPDATEはバッチ分割。** 一発UPDATEはロックで本番停止しうる。
- **進捗とエラーを記録するスクリプトに。** 途中で止まったときの再開可能性を確保。
- **ロールバック手順は紙に書く前にステージングで実行確認。** 「downがあるから安心」は罠。
- **削除は最後の最後。** カラム/テーブル削除を急がない。観測期間を十分に取る。
