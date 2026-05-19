# DBMS別 オンラインDDL / ロック影響リファレンス

`migration-plan` skill から参照される、DBMS 固有の DDL 適用ノウハウ。

## PostgreSQL

- `CREATE INDEX CONCURRENTLY` — 読み書きをブロックしない
- `ALTER TABLE ADD COLUMN ... DEFAULT ...` — PG11 以降は高速(メタデータのみ)
- `ALTER TABLE ALTER COLUMN ... TYPE ...` — テーブル全体のリライトが発生する場合あり、要注意
- `pg_repack` で長時間ロックを回避

## MySQL

- `ALGORITHM=INPLACE, LOCK=NONE` を明示
- `ALTER TABLE` のオンライン可否は変更内容次第。マニュアルで要確認
- 大規模変更は `pt-online-schema-change` / `gh-ost` を使用

## 共通の注意

- 5GB 以上のテーブルへの DDL は特に慎重に
- INDEX 作成はピーク時間外を選ぶ
- 長時間トランザクションの後ろに DDL が詰まると致命的
