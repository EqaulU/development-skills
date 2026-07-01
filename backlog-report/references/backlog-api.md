# Backlog API リファレンス(作業サマリ登録用)

`backlog-report` から作業サマリを Backlog に登録するための、検証済みエンドポイントとスクリプト。
公式: <https://developer.nulab.com/docs/backlog/>

## 共通

- **認証**: クエリパラメータ `apiKey`(個人設定 > API で発行)。
- **ベース URL**: `https://{BACKLOG_SPACE}/api/v2/...`
  - `BACKLOG_SPACE` はスペースのホスト名。ドメインは契約により異なる: `.backlog.com` / `.backlog.jp` / `.backlogtool.com`。フルホスト(例 `yourspace.backlog.jp`)を入れる。
- **Content-Type**: `application/x-www-form-urlencoded`。
- **Markdown 本文**は URL エンコードが必要。`curl --data-urlencode "content@ファイル"` を使うとファイルから読み込んでエンコードしてくれる(改行・記号の崩れを防げる)。

> 投稿は外部公開操作。実行前にユーザーへ要約を提示し承認を得る。初回は dry-run(コマンド表示のみ)を推奨。

## A. ドキュメント(既定)

### 作成 — `POST /api/v2/documents`

| パラメータ | 必須 | 説明 |
| :--- | :--- | :--- |
| `projectId` | ◯ | プロジェクトの数値 ID |
| `title` | | ドキュメント名(例 `作業ログサマリ 2026-06-24`) |
| `content` | | 本文(Markdown) |
| `emoji` | | 先頭に付く絵文字1文字 |
| `parentId` | | 親ドキュメント ID(配下に作る場合) |

```bash
# 要約は docs/worklog/summary-YYYY-MM-DD.md に書き出してある前提
DATE=$(date +%F)
SUMMARY="docs/worklog/summary-${DATE}.md"

curl -sS -X POST \
  "https://${BACKLOG_SPACE}/api/v2/documents?apiKey=${BACKLOG_API_KEY}" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "projectId=${BACKLOG_PROJECT_ID}" \
  --data-urlencode "title=作業ログサマリ ${DATE}" \
  --data-urlencode "content@${SUMMARY}" \
  ${BACKLOG_DOC_PARENT_ID:+-d "parentId=${BACKLOG_DOC_PARENT_ID}"}
```

**レスポンス(主なフィールド)**: `id`, `projectId`, `title`, `json`, `plain`, `statusId`, `emoji`, `created`, `updated`。
投稿後の URL は概ね `https://${BACKLOG_SPACE}/document/{PROJECT_KEY}/{id}` 形式(レスポンスの `id` を使う)。

### 削除 — `DELETE /api/v2/documents/{documentId}`

```bash
curl -sS -X DELETE \
  "https://${BACKLOG_SPACE}/api/v2/documents/${DOC_ID}?apiKey=${BACKLOG_API_KEY}"
```

> Document API は作成/削除が中心(本文更新エンドポイントは未確認)。**1日1ドキュメント新規作成**で運用する。同日に作り直すときは古い方を削除してから。育て続けたい(追記更新)なら Wiki を使う。

## B. Wiki(追記更新したい場合の代替)

### 作成 — `POST /api/v2/wikis`

| パラメータ | 必須 | 説明 |
| :--- | :--- | :--- |
| `projectId` | ◯ | プロジェクトの数値 ID |
| `name` | ◯ | ページ名(階層は `/` 区切り。例 `日報/2026-06-24`) |
| `content` | ◯ | 本文(Markdown) |
| `mailNotify` | | `true` で更新通知メール |

```bash
DATE=$(date +%F)
curl -sS -X POST \
  "https://${BACKLOG_SPACE}/api/v2/wikis?apiKey=${BACKLOG_API_KEY}" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "projectId=${BACKLOG_PROJECT_ID}" \
  --data-urlencode "name=日報/${DATE}" \
  --data-urlencode "content@docs/worklog/summary-${DATE}.md"
```

成功時 `201 CREATED`。レスポンスに `id`(= wikiId)。

### 更新 — `PATCH /api/v2/wikis/{wikiId}`

同じページを育てる場合。`name` / `content` / `mailNotify` を渡す。

```bash
curl -sS -X PATCH \
  "https://${BACKLOG_SPACE}/api/v2/wikis/${WIKI_ID}?apiKey=${BACKLOG_API_KEY}" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "content@docs/worklog/summary-${DATE}.md"
```

### 既存ページの探索 — `GET /api/v2/wikis?projectId=...`

二重作成を避けるため、投稿前に同名ページの有無を確認できる。

```bash
curl -sS "https://${BACKLOG_SPACE}/api/v2/wikis?apiKey=${BACKLOG_API_KEY}&projectId=${BACKLOG_PROJECT_ID}"
```

## PowerShell 版(Windows)

`curl` が使えない/エイリアスが `Invoke-WebRequest` の環境向け。本文はファイルから読み、フォームエンコードは `Invoke-RestMethod -Body` のハッシュテーブルに任せる。

```powershell
$Date    = Get-Date -Format 'yyyy-MM-dd'
$Summary = Get-Content "docs/worklog/summary-$Date.md" -Raw -Encoding utf8
$Uri     = "https://$env:BACKLOG_SPACE/api/v2/documents?apiKey=$env:BACKLOG_API_KEY"
$Body    = @{
  projectId = $env:BACKLOG_PROJECT_ID
  title     = "作業ログサマリ $Date"
  content   = $Summary
}
Invoke-RestMethod -Uri $Uri -Method Post -Body $Body `
  -ContentType 'application/x-www-form-urlencoded'
```

## エラー対処

| 症状 | 主な原因 | 対処 |
| :--- | :--- | :--- |
| `401 Unauthorized` | API キー誤り / 失効 | キーを再発行。`apiKey` がクエリに付いているか確認 |
| `404 Not Found` | スペースのドメイン違い / project ID 誤り | `.com`/`.jp`/`.backlogtool.com` を確認。`projectId` は**数値 ID**(プロジェクトキーではない) |
| `400 Bad Request` | 必須欠落 / 本文エンコード崩れ | `--data-urlencode "content@file"` を使う。Document は `projectId`、Wiki は `name`/`content` も必須 |
| 本文の改行・記号が壊れる | 手で本文を埋め込んだ | `-d "content=..."` ではなく `--data-urlencode "content@file"` でファイルから読む |
| 文字化け | ファイルが UTF-8 でない | 要約ファイルを UTF-8 で保存 |

## ID の調べ方

- **API キー**: Backlog 右上アバター > 個人設定 > API > 新しい API キーを発行。
- **プロジェクト ID(数値)**: `GET /api/v2/projects?apiKey=...` で一覧 → 対象の `id`。プロジェクトキー(英字)とは別物。
- **親ドキュメント ID**: 親にしたいドキュメントの URL 末尾、またはドキュメント一覧 API の `id`。
