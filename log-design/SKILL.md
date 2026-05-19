---
name: log-design
description: 構造化ログの設計をレビューする。ログレベル運用、PII漏洩、相関ID(trace_id/request_id)、コスト、検索性、保持期間、メトリクス分離。ログ設計・logger選定・構造化ログ・観測性の相談で使う。
---

# Log Design

ログの設計と実装をレビューする。「動いたか」を見るだけのログから、「問題が起きたときに追える」ログに引き上げる。

## ログレベルの運用

| レベル | 用途 | 例 |
| :--- | :--- | :--- |
| `FATAL` / `CRITICAL` | プロセス継続不能、即対応 | 起動時の必須設定欠落 |
| `ERROR` | 失敗、要調査(オンコール通知対象) | 外部API 5xx、DB接続失敗 |
| `WARN` | 異常だが処理は続行 | retry 中、deprecation 警告 |
| `INFO` | 正常な業務イベント | リクエスト受信、決済成功 |
| `DEBUG` | 開発・調査用詳細 | SQL クエリ、パラメータ |
| `TRACE` | 詳細トレース(通常 OFF) | 関数の入出力 |

本番のデフォルトは `INFO`。`DEBUG` 以下は環境変数で切替可能に。

## 構造化ログ

JSON や key-value で出力。人が読むだけでなく、検索・集計・アラートが可能になる。

```json
{
  "timestamp": "2026-05-19T03:21:45.123Z",
  "level": "ERROR",
  "service": "payment-api",
  "env": "production",
  "trace_id": "abc123",
  "request_id": "req-456",
  "user_id": "u_789",
  "event": "payment.failed",
  "amount": 1000,
  "currency": "JPY",
  "error": {
    "code": "card_declined",
    "message": "Your card was declined."
  }
}
```

## チェック観点

### 必須フィールド
- `timestamp`(ISO 8601 + ms + TZ)
- `level`
- `service` / `env`
- `trace_id` / `request_id`(分散システムなら必須)
- `event` または `message`

### 推奨フィールド(該当時)
- `user_id`(ハッシュ化 or 内部 ID、メールアドレス等の生 PII は NG)
- `tenant_id` / `org_id`
- `duration_ms`
- `error.code` / `error.message` / `error.stack`

### PII・機密情報
- パスワード、トークン、API キーをログに出さない
- メールアドレス、電話番号、クレカは原則マスクまたは除外
- 詳細は `pii-check` skill 参照

### コスト管理
- 高頻度イベントの全ログは破産の元(API リクエスト 1 件 = 1 ログ × QPS × 24h)
- サンプリング(エラーは 100%、INFO は 10% 等)
- 不要な `console.log` 残置を CI で検出

### 検索性・アラート
- メッセージは固定文字列 + 構造化フィールド(`logger.info('payment.failed', { amount })`)
- フリーテキスト連結(`${var} 失敗`)は検索しづらい
- アラート対象になるイベントは `event` を一意キーに

### 相関 ID
- HTTP ヘッダー(`X-Request-ID` / `traceparent`)で受け取り、無ければ生成
- 非同期処理(ジョブ、Webhook)にも伝搬
- 1 リクエストの全ログが trace_id で検索できること

### メトリクスとの分離
- 高頻度の数値(レイテンシ・カウント)はログではなくメトリクス(Prometheus 等)に
- ログは「特定イベントが起きたか」を見るためのもの

## 出力フォーマット

```markdown
## ログ設計レビュー結果

### 🔴 Critical
- **[PII 漏洩]** `auth/login.ts:42` で `logger.info('login', req.body)` がパスワードを記録
- **[コスト]** 全 HTTP リクエストで DEBUG ログを残しているため、月 ¥50 万の S3 ストレージコスト
- **[相関ID欠落]** 非同期ジョブで trace_id を伝搬していない → 障害追跡不可

### 🟡 改善推奨
- メッセージがフリーテキスト → `event` キーで構造化
- ERROR レベルにアラート未設定
- ログレベルが環境変数で切替できない

### 🟢 任意
- TRACE レベルの導入

### ✅ 確認済み
- 構造化 JSON 出力
- timestamp に TZ あり
```

## 原則

- **「動いた」より「追える」。** 障害時に時系列で追えるかが価値。
- **構造化を最優先。** フリーテキストは検索・集計できない。
- **相関 ID は最初から仕込む。** 後付けは全コード変更が必要になる。
- **PII を入れない。** 一度ログに残ると消すコストが極めて高い(バックアップ・SIEM 含む)。
- **コストを意識。** 「念のため全部 INFO」は本番で破産する。
- **ログとメトリクスとトレースを役割分担。** 全部をログで賄おうとしない(o11y 三本柱)。

## 関連スキル

- `pii-check` — ログに PII が混入していないかの詳細チェック
- `env-check` — ログレベルの環境変数管理
- `incident-postmortem` — 「ログが不足していた」を再発防止アクションに
