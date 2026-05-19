---
name: naming-review
description: 命名規則の統一性(camelCase / snake_case / PascalCase / kebab-case)、命名の明瞭さ・意図の表現、略語の一貫性、boolean接頭辞、定数のSCREAMING_SNAKE等をレビューする。ユーザーが「命名見て」「名前これでいい?」「変数名イケてる?」「リネームすべき?」「naming convention 統一して」と言ったとき、`/naming-review` と打ったときには必ずこのskillを使う。命名は後から直すコストが高いので、新規コードや設計レビュー時には積極的に発動すること。
---

# Naming Review

命名の一貫性と表現力をレビューする。

## チェック観点

### 形式の統一(言語/プロジェクト規約に依存)

| 種別 | 一般的な規約 |
| :--- | :--- |
| 変数・関数 (JS/TS/Java) | camelCase |
| 変数・関数 (Python/Ruby) | snake_case |
| クラス・型 | PascalCase |
| 定数 | SCREAMING_SNAKE_CASE |
| ファイル名 | kebab-case or snake_case (要プロジェクト規約) |
| 環境変数 | SCREAMING_SNAKE_CASE |
| URL/ルート | kebab-case |
| DBテーブル・カラム | snake_case (多数派) |

### 表現の明瞭さ

- **意図が伝わるか** — `data` `info` `tmp` `obj` `value` のような無意味名を避ける
- **嘘をついていないか** — `getUser()` が実は副作用ありで保存もする等
- **抽象化レベルが適切か** — `user` か `customer` か `account` か、ドメイン言語に揃える
- **長すぎ/短すぎ** — スコープが広いほど長く、狭いほど短く
- **マジックナンバー/文字列** — 名前付き定数化されているか

### 慣習

- **boolean** — `is`/`has`/`can`/`should` 接頭辞、否定形(`isNotXxx`)は避ける
- **配列/コレクション** — 複数形(`users`)
- **動詞接頭辞** — `get`(取得)/`fetch`(非同期取得)/`load`(初期化込み)/`build`(生成)を使い分け
- **対義語の対称性** — `start`/`stop`、`open`/`close`、`add`/`remove`(`add`/`delete` の混在は避ける)
- **略語** — プロジェクト内で同一の略し方に統一(`config` vs `cfg`、`url` vs `URL`)

## 出力フォーマット

```markdown
## 命名レビュー結果

### 🔴 統一性違反
- `user_id` と `userId` の混在
  - 該当: UserService.ts:23, UserRepo.ts:45
  - 推奨: プロジェクト規約に従い `userId` に統一

### 🟡 明瞭さの改善
- `data` → `userProfile` 推奨
  - 該当: handler.ts:80
  - 理由: 何のデータか分からない。文脈から `userProfile` のほうが意図が伝わる

### 🟢 改名候補(任意)
- 軽微な改善

### 👍 良い命名
- 特に意図が明確で参考にしたい命名
```

## 原則

- **既存規約を最優先。** プロジェクト内に多数派の規約があれば、それに合わせる(一貫性 > 個人の好み)。
- **改名は影響範囲とセットで提案。** 公開API/DBカラムの改名はコストが高い。互換性影響を必ず述べる。
- **辞書の存在を推奨。** よく出てくるドメイン用語があれば `docs/glossary.md` 化を勧める。
