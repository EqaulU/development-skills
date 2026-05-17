# System Development Skills (14個)

システム開発を支援する skill テンプレート集。
書く系・整形系の手順を Claude Code の skill としてまとめている。

それぞれの skill は `<skill-name>/SKILL.md` に格納されている。

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

### 🧪 テスト系
| skill | 概要 |
| :--- | :--- |
| `test-gen` | 関数・クラスのユニットテスト自動生成 |
| `e2e-scenario` | ユーザーストーリーから E2E シナリオ作成(Gherkin) |

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

### 🚀 デプロイ・運用系
| skill | 概要 |
| :--- | :--- |
| `deploy-checklist` | 本番デプロイ前のチェックリスト |
| `rollback-guide` | 障害時のロールバック手順生成 |
| `migration-plan` | DB スキーマ変更計画(Expand-Migrate-Contract) |

### 💬 コミュニケーション・管理系
| skill | 概要 |
| :--- | :--- |
| `commit-msg` | Conventional Commits 規約のコミットメッセージ生成 |
| `pr-description` | PR テンプレートに沿った説明文を自動作成 |
| `task-breakdown` | 大きなタスクをサブタスクに分解(INVEST 原則) |

## 関連リポジトリ

調査・分析の重い専門タスク(コードレビュー、セキュリティ監査、パフォーマンス診断、依存パッケージ監査、エッジケース洗い出し、SQL最適化、命名規則レビュー、環境変数チェック、ポストモーテム作成)は **sub-agent** として別リポジトリにまとめている。

→ [development-sub-agent](https://github.com/itourun2801/development-sub-agent)

## skill と sub-agent の使い分け

| | skill (このリポジトリ) | sub-agent |
| :--- | :--- | :--- |
| 配置 | `.claude/skills/<name>/SKILL.md` | `.claude/agents/<name>.md` |
| 実行コンテキスト | 同じ会話に手順書として注入 | 別コンテキストで独立実行 |
| 並列実行 | 不可 | 可 |
| 向くもの | **書く系・整形系**(手順を統一したい) | **読む系・調査系**(重い読み込みを隔離したい) |

## カスタマイズ指針

各 SKILL.md は「テンプレ」なので、以下を必要に応じて編集する:

1. **`description` フィールド** — チームで使う固有のキーワードを追加すると発動精度が上がる
2. **チェック観点** — プロジェクト固有のルール(社内コーディング規約、禁止 API 等)を追記
3. **出力フォーマット** — 既存の Issue/PR テンプレと合わせる
4. **参考リンク** — 社内ドキュメントへのリンクを足すと一貫性が出る
5. **言語** — 全 skill 日本語ベース。英語チームなら翻訳して使用

## skill 間の連携例

- `task-breakdown` → 分解したサブタスクを `pr-description` で PR 化
- `refactor` → 既存テスト不足なら `test-gen` で characterization test を先に
- `commit-msg` → リリース時に `changelog-writer` で集約
- `deploy-checklist` → 障害時は `rollback-guide` → 事後 `incident-postmortem` agent
- `migration-plan` → DB 変更前のレビューは `sql-optimize` agent と組み合わせる

## 次のステップ

実際に使ってみて、以下のような調整を推奨:

- 出力が冗長すぎる/簡素すぎる場合は、各 SKILL.md の出力フォーマット例を調整
- 特定の skill が意図と違うタイミングで発動する場合は `description` の文言を調整
- 複数 skill で重複する内容があれば、共通部分を `references/` に切り出して相互参照
