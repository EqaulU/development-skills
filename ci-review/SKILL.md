---
name: ci-review
description: CI/CD設定(GitHub Actions/GitLab CI/CircleCI等)をレビューする。実行時間、並列化、キャッシュ、シークレット取扱い、権限最小化、再現性、コスト観点。CI設定レビュー・パイプライン最適化・ワークフロー改善の依頼で使う。
---

# CI/CD Config Review

CI/CD 設定を実行時間・セキュリティ・コスト・信頼性の観点でレビューする。

## 対象ファイル

| プラットフォーム | パス |
| :--- | :--- |
| GitHub Actions | `.github/workflows/*.yml` |
| GitLab CI | `.gitlab-ci.yml` |
| CircleCI | `.circleci/config.yml` |
| Jenkins | `Jenkinsfile` |
| Buildkite | `.buildkite/pipeline.yml` |

## チェック観点

### セキュリティ
- **シークレット露出**
  - `echo "$SECRET"` でログに出していないか
  - PR からのワークフローで secrets が使えないことを意識(GitHub の `pull_request_target` は要警戒)
  - `secrets` を環境変数経由で渡す際の漏洩リスク
- **権限最小化**
  - GitHub Actions `permissions:` をジョブごとに最小化(デフォルトは write-all になりがち)
  - OIDC を活用してクラウド資格情報をハードコードしない
- **third-party action のピン留め**
  - `uses: foo/bar@v3` ではなく commit SHA に固定(supply chain 攻撃対策)
  - Dependabot で SHA も更新する

### 実行時間・並列化
- ジョブの並列化(test を shard、lint と test を並列等)
- 不要な再ビルド(キャッシュ未使用、`actions/cache` の key 設計)
- `if:` でスキップ可能なジョブ(docs のみ変更で全テスト走らせていないか)
- matrix の網羅範囲(全 OS × 全 Node バージョンは過剰なことも)

### キャッシュ
- node_modules / pnpm-store / pip / cargo / gradle のキャッシュ
- key 設計(lockfile hash がベース、fallback restore-keys を活用)
- Docker レイヤキャッシュ(`type=gha` / Buildx)

### 信頼性・再現性
- pin されていないツールバージョン(`actions/setup-node@v4` だけでなく `node-version` も pin)
- `latest` イメージタグ(`ubuntu-latest` は許容、自前イメージは要 pin)
- フレーキーテストのリトライ戦略
- タイムアウト設定(`timeout-minutes:`)

### コスト
- 長時間ジョブ(self-hosted ランナーか、GitHub-hosted のままか判断)
- 不要な PR ごとの実行(`paths:` / `paths-ignore:` で絞れないか)
- 大規模 matrix の必要性
- artifact 保持期間(デフォルト 90 日は長すぎることが多い)

### デプロイの安全性
- main ブランチ保護(承認必須、CI 緑必須)
- 環境ごとの承認(production には `environment:` + reviewer)
- ロールバック可能な仕組み(タグベース、フィーチャーフラグ連携)

## 出力フォーマット

```markdown
## CI/CD 設定レビュー結果

### 🔴 Critical
- **[セキュリティ]** `.github/workflows/deploy.yml` で `permissions:` 未指定 → write-all
  - 修正: 必要な権限のみ列挙(例: `contents: read`, `id-token: write`)
- **[セキュリティ]** `uses: some-org/action@main` で SHA 未固定
  - 修正: commit SHA に pin、Dependabot で SHA 更新

### 🟡 改善推奨
- **[実行時間]** test ジョブで node_modules キャッシュ未使用 → ~3 分短縮可能
- **[実行時間]** lint と test が直列実行 → 並列化で ~5 分短縮
- **[コスト]** matrix で全 OS × Node 16/18/20/22 → 18/22 のみで十分では

### 🟢 任意
- artifact 保持期間 30 日に短縮

### ✅ 確認済み
- secrets のログ露出なし
- production デプロイは reviewer 必須
```

## 原則

- **権限はデフォルト deny。** `permissions:` を明示的に最小化。
- **third-party は SHA pin。** タグは書き換え可能。
- **キャッシュキーは過不足なく。** lockfile が変わったら invalidate、コードが変わっただけなら hit。
- **時間とコストはトレードオフ。** 並列化で時間を短縮、ジョブを絞ってコストを抑える。両方追わない場合は短縮優先。
- **失敗時の通知設計。** main で落ちたら誰がどう気づくかを明確に。Slack/email 通知の owner を決める。

## 関連スキル

- `deploy-checklist` — CI が緑であることはリリース前提条件
- `dependency-audit` — CI で自動スキャンを組み込む
- `security-check` — CI 設定自体のセキュリティレビューと併用
