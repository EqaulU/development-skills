---
name: commit-msg
description: diff/ステージ変更からConventional Commits規約に沿ったコミットメッセージを生成する(type/scope/件名/本文/Breaking Changes/Issue参照)。コミット文作成の依頼で使う。
---

# Commit Message

変更内容からコミットメッセージを生成する。デフォルトは Conventional Commits 規約。

## 入力情報の取得

開始時に、以下のいずれかから変更を把握する:

```bash
git diff --staged           # ステージされた変更
git diff                    # 未ステージ変更
git diff HEAD~1             # 直近1コミット
```

## Conventional Commits フォーマット

```
<type>(<scope>): <subject>

<body>

<footer>
```

### type

| type | 用途 |
| :--- | :--- |
| `feat` | 新機能 |
| `fix` | バグ修正 |
| `docs` | ドキュメントのみ |
| `style` | 整形(挙動非変更、空白・セミコロン等) |
| `refactor` | 挙動変更なしの構造改善 |
| `perf` | パフォーマンス改善 |
| `test` | テスト追加・修正 |
| `build` | ビルド設定、依存パッケージ |
| `ci` | CI設定 |
| `chore` | 上記いずれにも該当しない雑作業 |
| `revert` | 過去コミットの取り消し |

### scope(任意)

影響範囲(`auth`, `api`, `ui`, `db`, モジュール名等)。プロジェクトで定義された scope があれば従う。

### subject(必須)

- 50字以内
- 命令形・現在形(`add` であって `added` ではない)
- 末尾にピリオドを付けない
- 小文字始まり(プロジェクト規約に従う)
- 日本語OK(プロジェクト言語に従う)

### body(任意)

- なぜ変更したか(Whatではなく Why)
- 72字で折り返し
- 空行を挟んで subject と分離

### footer(任意)

- 関連Issue: `Refs: #123`, `Closes: #123`
- Breaking Change: `BREAKING CHANGE: 説明`
- 共著: `Co-authored-by: Name <email>`

## 例

### 例1: 機能追加(シンプル)

```
feat(auth): add password reset endpoint
```

### 例2: バグ修正(本文付き)

```
fix(payment): handle null currency in invoice calculation

Stripe API は通貨が指定されないリクエストに対して
currency フィールドを返さないことがある。これにより
請求書生成時に TypeError が発生していた。

Refs: #234
```

### 例3: 破壊的変更

```
feat(api)!: change user response format to camelCase

すべてのJSONレスポンスフィールドを snake_case から
camelCase に統一する。

BREAKING CHANGE: API consumers must update field
accesses (e.g. `user_id` → `userId`). 移行ガイドは
docs/migration-v2.md を参照。

Refs: #500
```

### 例4: 複数ファイルだが本質は1つ

```
refactor: extract validation logic to shared module

User/Order/Product それぞれに散在していたバリデーション
ロジックを `lib/validators` に集約。挙動は変更なし。
```

## 変更内容 → type 判定の指針

| diffの特徴 | 推奨 type |
| :--- | :--- |
| 新しい関数・エンドポイント・画面の追加 | `feat` |
| 既存コードのバグ修正(挙動の誤り) | `fix` |
| import順、フォーマッタ適用、空白のみ | `style` |
| 関数分割・改名・移動のみ、挙動変更なし | `refactor` |
| クエリ最適化、キャッシュ追加 | `perf` |
| `*.test.ts` のみの変更 | `test` |
| `package.json`, `pnpm-lock`, `Dockerfile` | `build` |
| `.github/workflows/`, `.gitlab-ci.yml` | `ci` |
| `README.md`, `*.md` のみ | `docs` |

複数のtypeが混在する場合は、コミット分割を提案する。

## 原則

- **subject は「このコミットを適用すると〜になる」と読める文に。** 例: `fix login bug` を読んで「ログインバグを修正する」が自然。
- **「なぜ」を body に。** diff を見れば「何が変わったか」は分かる。動機・経緯・代替案を選ばなかった理由を書く。
- **複数のtypeが混ざる diff はコミット分割を提案。** `feat` と `refactor` を1コミットに入れない。
- **既存のリポジトリ規約を優先。** Conventional Commits を使っていないリポジトリで強制しない。`git log --oneline -20` で慣習を確認。
