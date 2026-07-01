# System Development Skills (41個)

システム開発を支援する skill テンプレート集。
それぞれの skill は `<skill-name>/SKILL.md` に格納されている。
長いテンプレートや DBMS 固有の参照情報は `<skill-name>/references/*.md` に分離(Progressive Disclosure)。

## インストール

このリポジトリの各ディレクトリは Claude Code の skill としてそのまま使える構成になっている。
`~/.claude/skills/` 配下にコピーするだけで有効化される。

### ユーザー全体に入れる場合

```bash
git clone https://github.com/itourun2801/development-skills.git
cd development-skills
mkdir -p ~/.claude/skills
cp -r */ ~/.claude/skills/
```

### プロジェクト単位で入れる場合

```bash
mkdir -p .claude/skills
cp -r /path/to/development-skills/commit-msg .claude/skills/
```

### 個別にインストール

```bash
cp -r commit-msg ~/.claude/skills/
```

## 使い方

1. 各 `SKILL.md` をプロジェクト固有の用語・規約に合わせて編集
2. 上記手順でインストール
3. 該当キーワード(skill description に記載)を含むメッセージで自動発動、または明示的に呼び出し

## 全 skill 一覧

### 🧭 ワークフロー / オーケストレーション
| skill | 概要 |
| :--- | :--- |
| `dev-workflow` | 要件定義→設計→製造→レビュー→単体/結合/総合テストを各スキルを呼びゲート付きで駆動 |

### 📊 見積もり・計画系
| skill | 概要 |
| :--- | :--- |
| `effort-estimate` | 開発工数をストーリーポイント/三点見積もり(PERT)/類推/ボトムアップ等で見積もり、前提・レンジ・バッファを明示 |
| `cost-estimate` | 工数・数量から商用の見積書(明細・単価・人月・値引き・消費税・前提条件・スコープ外)を作成 |

### 🏗️ 要件・設計系
| skill | 概要 |
| :--- | :--- |
| `requirements` | 要望を検証可能な要件・ユーザーストーリー・受入基準・優先度(MoSCoW)に整理 |
| `design-doc` | 実装前の設計書/RFC(背景・目的・非目的・代替案・段階移行・リスク) |
| `adr` | 設計判断1件をADRとして記録(文脈・決定・選択肢・結果) |
| `api-design` | 実装前のAPI契約設計(リソース・命名・エラーモデル・バージョニング) |
| `data-model` | 実装前のデータモデル設計(ER・正規化・キー・インデックス戦略) |
| `threat-model` | 設計時のセキュリティ脅威分析(STRIDE・攻撃面・緩和策) |

### 🔍 コードレビュー系
| skill | 概要 |
| :--- | :--- |
| `code-review` | 可読性・セキュリティ・パフォーマンス等の統一観点でレビュー |
| `security-check` | OWASP Top 10 / CWE 観点で脆弱性を洗い出し |
| `perf-review` | N+1、不要ループ、DB クエリ効率、メモリリーク等を指摘 |
| `naming-review` | 命名規則(camelCase/snake_case 等)の統一チェック |
| `a11y-review` | アクセシビリティ(WCAG 2.1/2.2 AA)のレビュー |
| `pii-check` | PII・機密データの取扱い、ログ・URL・分析イベント漏洩検出 |

### 🧪 テスト系
| skill | 概要 |
| :--- | :--- |
| `test-gen` | 関数・クラスのユニットテスト自動生成 |
| `integration-test` | コンポーネント境界の結合テスト(API↔DB、DAO、外部APIスタブ) |
| `e2e-scenario` | ユーザーストーリーから E2E シナリオ作成(Gherkin) |
| `edge-case-finder` | 境界値・異常系・例外パターンを系統的に列挙 |

### 📝 ドキュメント系
| skill | 概要 |
| :--- | :--- |
| `doc-gen` | JSDoc / TSDoc / docstring 等の自動生成 |
| `readme-update` | 変更内容に合わせて README を更新 |
| `changelog-writer` | コミット履歴から CHANGELOG 生成(Keep a Changelog 形式) |
| `api-doc` | エンドポイントから OpenAPI 仕様書ドラフトを生成 |

### 🔧 リファクタリング系
| skill | 概要 |
| :--- | :--- |
| `refactor` | 重複排除・関数分割・責務整理(挙動保持) |
| `type-strengthen` | TypeScript 型の強化(any 排除・型ガード等) |
| `sql-optimize` | スロークエリ改善・INDEX 設計レビュー |

### 🚀 デプロイ・運用系
| skill | 概要 |
| :--- | :--- |
| `deploy-checklist` | 本番デプロイ前のチェックリスト |
| `rollback-guide` | 障害時のロールバック手順生成 |
| `env-check` | 環境変数の漏れ・差分・シークレット混入検出 |
| `migration-plan` | DB スキーマ変更計画(Expand-Migrate-Contract) |
| `ci-review` | CI/CD 設定(GitHub Actions 等)のレビュー |
| `log-design` | 構造化ログ設計、レベル運用、PII 対策、コスト管理 |

### 💬 コミュニケーション・管理系
| skill | 概要 |
| :--- | :--- |
| `commit-msg` | Conventional Commits 規約のコミットメッセージ生成 |
| `pr-description` | PR テンプレートに沿った説明文を自動作成 |
| `task-breakdown` | 大きなタスクをサブタスクに分解(INVEST 原則) |
| `incident-postmortem` | 障害事後分析(Blameless Postmortem) |
| `git-history` | git 履歴整理(rebase/squash/cherry-pick) |
| `work-log` | 作業ごとの記録(やったこと・判断・詰まり・次アクション)を Markdown で残す |
| `backlog-report` | work-log の当日分を要約し Backlog(Nulab)のドキュメント/Wiki に API 登録 |

### 🌐 国際化・運用
| skill | 概要 |
| :--- | :--- |
| `i18n-check` | i18n/l10n の漏れ検出(ハードコード、複数形、日付/通貨等) |
| `dependency-audit` | 依存パッケージの脆弱性・ライセンス・メンテ状況スキャン |

## 関連リポジトリ

調査・分析の重いタスクは、**sub-agent 形式**でも入手可能。並列実行や別コンテキスト隔離が必要なときに使う。

→ [development-sub-agent](https://github.com/itourun2801/development-sub-agent)

ただし sub-agent は起動コストが大きくトークン消費が重いので、通常はこのリポジトリ(skill 形式)の使用を推奨する。

## skill と sub-agent の使い分け

| | skill (このリポジトリ) | sub-agent |
| :--- | :--- | :--- |
| 配置 | `.claude/skills/<name>/SKILL.md` | `.claude/agents/<name>.md` |
| 実行コンテキスト | 同じ会話に手順書として注入 | 別コンテキストで独立実行 |
| 並列実行 | 不可 | 可 |
| トークン消費 | **軽い** | **重い**(起動・再読み込みコスト) |
| 向くシーン | 日常的な利用全般 | 大規模並列調査・コンテキスト隔離が必要なとき |

## Progressive Disclosure

長いテンプレート・DBMS 固有の詳細・出力例などは `<skill>/references/*.md` に分離している。
SKILL.md 本体は常にロードされるが、references/ は Claude が必要と判断したときだけ読み込む。

例:
- `dev-workflow/references/playbook.md` — フェーズ別ゲート・規模別調整・戻り方
- `effort-estimate/references/methods.md` — 見積もり手法カタログ・PERT計算・アンチパターン
- `cost-estimate/references/template.md` — 見積書フルテンプレート・端数/税/値引きの扱い
- `requirements/references/checklist.md` — NFRカテゴリ・質問バンク・受入基準の書き方
- `design-doc/references/template.md` — 設計書フルテンプレート
- `api-design/references/conventions.md` — HTTP ステータス/ページング/エラー規約
- `data-model/references/patterns.md` — 正規形・モデリングパターン・アンチパターン
- `threat-model/references/stride.md` — STRIDE 脅威×緩和策カタログ
- `integration-test/references/harness-setup.md` — テストDB・スタブ・FW別ハーネス設定
- `migration-plan/references/expand-migrate-contract.md` — フェーズ別 SQL 例
- `migration-plan/references/dbms-online-ddl.md` — PostgreSQL/MySQL のオンライン DDL
- `sql-optimize/references/explain-cheatsheet.md` — EXPLAIN 解読チートシート
- `incident-postmortem/references/template.md` — 完全テンプレート
- `commit-msg/references/examples.md` — Conventional Commits 例集
- `pr-description/references/templates.md` — PR 種別別テンプレート
- `rollback-guide/references/full-procedure.md` — フル手順テンプレート
- `api-doc/references/openapi-example.md` — OpenAPI 出力例
- `code-review/references/project-rules.md` — プロジェクト固有ルール雛形
- `backlog-report/references/backlog-api.md` — Backlog Document/Wiki API のエンドポイント・curl/PowerShell・エラー対処

## カスタマイズ指針

各 SKILL.md は「テンプレ」なので、以下を必要に応じて編集する:

1. **`description` フィールド** — チームで使う固有のキーワードを追加すると発動精度が上がる
2. **チェック観点** — プロジェクト固有のルール(社内コーディング規約、禁止 API 等)を追記
3. **出力フォーマット** — 既存の Issue/PR テンプレと合わせる
4. **参考リンク** — 社内ドキュメントへのリンクを足すと一貫性が出る
5. **言語** — 全 skill 日本語ベース。英語チームなら翻訳して使用
6. **references/** — プロジェクト固有のテンプレートを追加できる

## skill 間の連携例

- `dev-workflow` → 全フェーズの司令塔。下記の各連携を要件→設計→実装→レビュー→テストの順に駆動
- `requirements` → `design-doc`(どう作るか)へ。受入基準は `e2e-scenario` の元ネタ、ストーリーは `task-breakdown` で分解
- `design-doc` → 個別の重要判断を `adr` で記録 → `task-breakdown` で実装分解
- `design-doc` → `api-design` / `data-model` で API 契約・スキーマを具体化 → 実装
- `api-design` → 実装後に `api-doc` で OpenAPI 化
- `data-model` → `migration-plan` で安全に反映 → 運用後 `sql-optimize` で INDEX 見直し
- `threat-model` → `security-check`(コード)/ `pii-check` / `log-design` でドリルダウン
- `task-breakdown` → 分解したサブタスクを `effort-estimate` で工数見積もり → `cost-estimate` で金額の見積書に換算
- `requirements` → `effort-estimate`(前提が固まるほど見積もり精度が上がる)
- `task-breakdown` → 分解したサブタスクを `pr-description` で PR 化
- `test-gen`(ユニット)→ `integration-test`(結合)→ `e2e-scenario`(E2E)でテストピラミッドを構成
- `data-model` / `api-design` の設計が噛み合うかを `integration-test` で裏取り
- `refactor` → 既存テスト不足なら `test-gen` で characterization test を先に
- `code-review` の指摘 → `security-check` / `perf-review` / `naming-review` / `a11y-review` / `pii-check` でドリルダウン
- `commit-msg` → リリース時に `changelog-writer` で集約
- `deploy-checklist` → 障害時は `rollback-guide` → 事後 `incident-postmortem`
- `migration-plan` → `sql-optimize` と組み合わせて INDEX 設計レビュー
- `pii-check` ↔ `log-design` — ログへの PII 漏洩を相互チェック
- `i18n-check` ↔ `a11y-review` — `lang`、文字方向、コントラストは両方の関心
- `git-history` → `commit-msg` で再生成、`pr-description` で再構成
- `work-log` → 日々の作業の生ログ。重要判断は `adr`、リリース変更は `changelog-writer`、障害対応は `incident-postmortem` に昇華
- `work-log` → `backlog-report` で当日分を要約し Backlog のドキュメント/Wiki に登録(投稿前に `pii-check` / `env-check`)

## 次のステップ

実際に使ってみて、以下のような調整を推奨:

- 出力が冗長すぎる/簡素すぎる場合は、各 SKILL.md の出力フォーマット例を調整
- 特定の skill が意図と違うタイミングで発動する場合は `description` の文言を調整
- 複数 skill で重複する内容があれば、共通部分を `references/` に切り出して相互参照
