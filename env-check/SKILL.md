---
name: env-check
description: 環境変数の設定漏れ・環境間差分・.env.example同期・シークレットのコード混入を検出する。環境変数チェック・設定差分確認の依頼で使う。
---

# Environment Check

環境変数・設定の整合性を確認する。

## チェック項目

### 1. `.env.example` とコード参照の整合
- コード内で参照されている環境変数(`process.env.XXX` / `os.environ['XXX']` / `ENV['XXX']` 等)を全て列挙
- `.env.example` に記載されているキーと突き合わせ
- **コードで使うが example に無い** → 設定漏れリスク(本番でundefined)
- **example にあるがコードで使わない** → 死んだ設定(削除候補)

### 2. 環境間の差分
- `dev` / `staging` / `prod` で必要なキーが揃っているか
- デフォルト値が環境ごとに適切か(例: `DEBUG=true` が本番に紛れていないか)
- URL/接続先が本番でlocalhostになっていないか

### 3. シークレットのコード混入
- ハードコードされたAPIキー、パスワード、トークン
- gitに追跡されている `.env` ファイル
- ログ出力に環境変数の値がそのまま出ていないか

### 4. 型・形式の妥当性
- 数値であるべき値が文字列のまま使われていないか(`PORT=3000` を `Number(process.env.PORT)` 等)
- URLの末尾スラッシュの一貫性
- boolean(`"true"` / `"false"` 文字列)の扱い

### 5. 命名と運用
- SCREAMING_SNAKE_CASE 統一
- プレフィックスでグループ化(`DB_*`, `REDIS_*`, `AUTH_*`)
- 本番でのシークレット管理(Vault / AWS Secrets Manager / GCP Secret Manager 等)に登録済みか

## 検出スクリプト例

```bash
# コードで参照されている環境変数を抽出(JS/TS)
grep -rEh "process\.env\.[A-Z_][A-Z0-9_]*" src/ \
  | grep -oE "process\.env\.[A-Z_][A-Z0-9_]*" \
  | sort -u \
  | sed 's/process\.env\.//'

# .env.example のキーを抽出
grep -E "^[A-Z_]+=" .env.example | cut -d= -f1 | sort -u

# 差分
diff <(...) <(...)
```

```bash
# Python
grep -rEh "os\.environ\.get\(['\"][A-Z_]+" src/ \
  | grep -oE "['\"][A-Z_]+['\"]" \
  | tr -d "'\"" | sort -u
```

```bash
# シークレット候補の検出(誤検知前提でレビューする)
git grep -E "(api[_-]?key|secret|token|password)\s*[:=]\s*['\"][^'\"]{8,}" -- ':!*.md' ':!*.example'
```

## 出力フォーマット

```markdown
## 環境変数チェック結果

### 🔴 設定漏れ(コードで参照 / exampleに無し)
| 変数名 | 参照箇所 | 推奨デフォルト |
| :--- | :--- | :--- |
| `STRIPE_WEBHOOK_SECRET` | services/payment.ts:42 | (シークレット、本番のみ設定) |

### 🟡 死んだ設定(exampleにあるが未使用)
- `OLD_API_URL` — どこからも参照されていない。削除候補

### 🔴 シークレット混入疑い
- `src/lib/aws.ts:12` に `accessKeyId: 'AKIA...'` のハードコードあり
  - 対応: 直ちに環境変数化 + 該当キーをローテーション

### 🟡 環境間差分
- staging に `SENTRY_DSN` 未設定 → エラー監視が効かない可能性
- prod に `DEBUG=true` → 詳細エラーが露出するリスク

### ✅ 確認済み(OK)
- N個のキーが整合
```

## 原則

- **シークレットを発見したら速やかにローテーション提案。** 既にgit履歴に入っていたら、変数化だけでは不十分。
- **`.env.example` は本物のシークレットを書かない。** プレースホルダ(`xxx` / `your-key-here`)に留める。
- **存在チェックと型チェックは起動時に一括で。** `zod`/`envsafe` 等のバリデータ導入も検討。
- **環境ごとの設定差分は最小に。** 設定の差分が大きいほど環境固有バグが増える。
