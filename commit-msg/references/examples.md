# コミットメッセージ例集

`commit-msg` skill から参照される、Conventional Commits の具体例。

## 例1: 機能追加(シンプル)

```
feat(auth): add password reset endpoint
```

## 例2: バグ修正(本文付き)

```
fix(payment): handle null currency in invoice calculation

Stripe API は通貨が指定されないリクエストに対して
currency フィールドを返さないことがある。これにより
請求書生成時に TypeError が発生していた。

Refs: #234
```

## 例3: 破壊的変更

```
feat(api)!: change user response format to camelCase

すべての JSON レスポンスフィールドを snake_case から
camelCase に統一する。

BREAKING CHANGE: API consumers must update field
accesses (e.g. `user_id` → `userId`). 移行ガイドは
docs/migration-v2.md を参照。

Refs: #500
```

## 例4: 複数ファイルだが本質は 1 つ

```
refactor: extract validation logic to shared module

User/Order/Product それぞれに散在していたバリデーション
ロジックを `lib/validators` に集約。挙動は変更なし。
```

## 例5: パフォーマンス改善

```
perf(search): add composite index on (user_id, created_at)

検索一覧 API が p95 で 1.2s → 80ms に短縮。
EXPLAIN ANALYZE の結果は #456 参照。

Refs: #455
```

## 例6: revert

```
revert: feat(auth): add password reset endpoint

This reverts commit abc1234.
プロダクション環境でメール送信が失敗するため一旦差し戻し。
原因調査後に再実装する。

Refs: #501
```

## 変更内容 → type 判定の指針

| diff の特徴 | 推奨 type |
| :--- | :--- |
| 新しい関数・エンドポイント・画面の追加 | `feat` |
| 既存コードのバグ修正(挙動の誤り) | `fix` |
| import 順、フォーマッタ適用、空白のみ | `style` |
| 関数分割・改名・移動のみ、挙動変更なし | `refactor` |
| クエリ最適化、キャッシュ追加 | `perf` |
| `*.test.ts` のみの変更 | `test` |
| `package.json`, `pnpm-lock`, `Dockerfile` | `build` |
| `.github/workflows/`, `.gitlab-ci.yml` | `ci` |
| `README.md`, `*.md` のみ | `docs` |

複数の type が混在する場合は、コミット分割を提案する。
