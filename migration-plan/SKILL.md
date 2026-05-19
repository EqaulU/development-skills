---
name: migration-plan
description: DBスキーマ変更・データ移行の計画を作成する(Expand-Migrate-Contractパターン、オンラインDDL、ロールバック手順)。マイグレーション・スキーマ変更・ゼロダウンタイムの依頼で使う。
---

# Migration Plan

DB スキーマ変更とデータ移行の計画を立てる。

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
| **後方互換(Additive)** | 低 | カラム追加(NULLABLE)、INDEX 追加、テーブル追加 |
| **両方向互換(Compatible)** | 中 | カラム改名(エイリアス併用)、NOT NULL 化(既存データ充足済み) |
| **破壊的(Breaking)** | 高 | カラム削除、型変更、NOT NULL 化(NULL 残存)、テーブル削除 |

破壊的変更は **Expand-Migrate-Contract** パターンで段階分割する。

## Expand-Migrate-Contract パターン(概要)

4 フェーズで進める:

1. **Expand** — 新カラム/テーブル追加(NULLABLE で後方互換)+ アプリは Dual Write 化
2. **Backfill** — 既存データをバッチで埋める
3. **Migrate** — 読み取り側を新カラムに切替(feature flag でカナリア推奨)
4. **Contract** — 観測期間後に旧カラム削除

具体的な SQL 例と出力テンプレートは [references/expand-migrate-contract.md](references/expand-migrate-contract.md) を参照。

## オンライン DDL / ロック影響

DBMS 固有のオンラインDDL対応 (PostgreSQL の `CREATE INDEX CONCURRENTLY`、MySQL の `ALGORITHM=INPLACE` 等) は [references/dbms-online-ddl.md](references/dbms-online-ddl.md) を参照。

共通の注意点:
- 5GB 以上のテーブルへの DDL は特に慎重に
- INDEX 作成はピーク時間外
- 長時間トランザクションの後ろに DDL が詰まると致命的

## 原則

- **本番DBの変更はリハーサルが必須。** ステージングで本番相当のデータ量でテストする。
- **後方互換を最大限維持する設計。** 「アプリとDBが一瞬不一致になる時間帯」が出ないように。
- **大規模UPDATEはバッチ分割。** 一発UPDATEはロックで本番停止しうる。
- **進捗とエラーを記録するスクリプトに。** 途中で止まったときの再開可能性を確保。
- **ロールバック手順は紙に書く前にステージングで実行確認。** 「downがあるから安心」は罠。
- **削除は最後の最後。** カラム/テーブル削除を急がない。観測期間を十分に取る。

## 関連スキル

- `sql-optimize` — 新スキーマ・新クエリの INDEX 設計とパフォーマンス検証
- `rollback-guide` — Phase 別の切り戻し手順を整理
- `deploy-checklist` — マイグレーションを含むリリースのチェックリスト
