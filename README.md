# System Development Skills (23個)

システム開発を支援するskillsテンプレート集。
それぞれの skill は `<skill-name>/SKILL.md` に格納されている。

## インストール

このリポジトリの各ディレクトリは Claude Code の skill としてそのまま使える構成になっている。
`~/.claude/skills/` 配下にコピーするだけで有効化される。

### ユーザー全体に入れる場合

```bash
git clone https://github.com/itourun2801/development-skills.git
cd development-skills
# 23個まとめてインストール
mkdir -p ~/.claude/skills
cp -r */ ~/.claude/skills/
```

### プロジェクト単位で入れる場合

```bash
# プロジェクトルートで
mkdir -p .claude/skills
cp -r /path/to/development-skills/code-review .claude/skills/
```

### 個別にインストール

```bash
cp -r code-review ~/.claude/skills/
```

## 使い方

1. 各 `SKILL.md` をプロジェクト固有の用語・規約に合わせて編集
2. 上記手順でインストール
3. 該当キーワード(skill descriptionに記載)を含むメッセージで自動発動、または明示的に呼び出し

## 全skill一覧

### 🔍 コードレビュー系
| skill | 概要 |
| :--- | :--- |
| `code-review` | 可読性・セキュリティ・パフォーマンス等の統一観点でレビュー |
| `security-check` | OWASP Top 10 / CWE 観点で脆弱性を洗い出し |
| `perf-review` | N+1、不要ループ、DBクエリ効率、メモリリーク等を指摘 |
| `naming-review` | 命名規則(camelCase/snake_case等)の統一チェック |

### 🧪 テスト系
| skill | 概要 |
| :--- | :--- |
| `test-gen` | 関数・クラスのユニットテスト自動生成 |
| `e2e-scenario` | ユーザーストーリーからE2Eシナリオ作成(Gherkin) |
| `edge-case-finder` | 境界値・異常系・例外パターンを系統的に列挙 |

### 📝 ドキュメント系
| skill | 概要 |
| :--- | :--- |
| `doc-gen` | JSDoc / TSDoc / docstring 等の自動生成 |
| `readme-update` | 変更内容に合わせてREADMEを更新 |
| `changelog-writer` | コミット履歴からCHANGELOG生成(Keep a Changelog形式) |
| `api-doc` | エンドポイントからOpenAPI仕様書ドラフトを生成 |

### 🔧 リファクタリング系
| skill | 概要 |
| :--- | :--- |
| `refactor` | 重複排除・関数分割・責務整理(挙動保持) |
| `type-strengthen` | TypeScript型の強化(any排除・型ガード等) |
| `sql-optimize` | スロークエリ改善・INDEX設計レビュー |

### 🚀 デプロイ・運用系
| skill | 概要 |
| :--- | :--- |
| `deploy-checklist` | 本番デプロイ前のチェックリスト |
| `rollback-guide` | 障害時のロールバック手順生成 |
| `env-check` | 環境変数の漏れ・差分・シークレット混入検出 |

### 💬 コミュニケーション・管理系
| skill | 概要 |
| :--- | :--- |
| `commit-msg` | Conventional Commits 規約のコミットメッセージ生成 |
| `pr-description` | PRテンプレートに沿った説明文を自動作成 |
| `task-breakdown` | 大きなタスクをサブタスクに分解(INVEST原則) |

### ➕ 追加提案(運用上有用)
| skill | 概要 |
| :--- | :--- |
| `dependency-audit` | 依存パッケージの脆弱性・ライセンス・メンテ状況スキャン |
| `migration-plan` | DBスキーマ変更計画(Expand-Migrate-Contract) |
| `incident-postmortem` | 障害事後分析(Blameless Postmortem) |

## カスタマイズ指針

各SKILL.mdは「テンプレ」なので、以下を必要に応じて編集する:

1. **`description` フィールド** — チームで使う固有のキーワードを追加すると発動精度が上がる
2. **チェック観点** — プロジェクト固有のルール(社内コーディング規約、禁止API等)を追記
3. **出力フォーマット** — 既存のIssue/PRテンプレと合わせる
4. **参考リンク** — 社内ドキュメントへのリンクを足すと一貫性が出る
5. **言語** — 全skill日本語ベース。英語チームなら翻訳して使用

## skill間の連携例

- `task-breakdown` → 分解したサブタスクを `pr-description` でPR化
- `refactor` → 既存テスト不足なら `test-gen` で characterization test を先に
- `code-review` の指摘 → `security-check` / `perf-review` / `naming-review` でドリルダウン
- `commit-msg` → リリース時に `changelog-writer` で集約
- `deploy-checklist` → 障害時は `rollback-guide` → 事後 `incident-postmortem`

## 次のステップ

実際に使ってみて、以下のような調整を推奨:

- 出力が冗長すぎる/簡素すぎる場合は、各SKILL.mdの出力フォーマット例を調整
- 特定のskillが意図と違うタイミングで発動する場合は `description` の文言を調整
- 複数skillで重複する内容があれば、共通部分を `references/` に切り出して相互参照
