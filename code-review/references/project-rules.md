# Project-Specific Review Rules (テンプレート)

`code-review` skill から参照されるプロジェクト固有ルール。
このファイルは**テンプレート**なので、各プロジェクトでコピーして使うこと。

## 使い方

1. このファイルを自分のプロジェクトの `.claude/skills/code-review/references/project-rules.md` にコピー
2. 下記項目を自プロジェクトの規約で埋める
3. SKILL.md 本体はそのままで、Claude がレビュー時に参照する

## 命名規則

- 変数/関数: <camelCase / snake_case / etc>
- クラス/型: <PascalCase>
- 定数: <SCREAMING_SNAKE_CASE>
- ファイル名: <kebab-case / snake_case>

## レイヤー構成

- `controllers/` → `services/` → `repositories/` の依存方向のみ許可
- ドメインロジックは `services/` 配下に
- ...

## 禁止 API / パターン

- `console.log` の残置 NG(`logger.info` を使う)
- `any` / `as` の新規追加 NG(`unknown` + 型ガード)
- 直接 `process.env.X` を参照しない(`config.X` 経由)
- ...

## 必須対応事項

- 公開 API 変更時は `docs/api.md` も更新
- DB スキーマ変更時は `migration-plan` skill を併用
- 新規エンドポイントには認可テストを追加
- ...

## レビュー時の追加チェックポイント

- (プロジェクト固有の観点をここに)
