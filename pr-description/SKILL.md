---
name: pr-description
description: 変更内容からPR/MR説明文(概要・背景・変更内容・テスト・レビュー観点・チェックリスト)を生成する。PR/MR説明作成の依頼で使う。
---

# Pull Request Description

変更内容から PR 説明文を生成する。

## 入力情報

開始時に把握する:

- **変更内容** — `git log` / `git diff` / 説明
- **関連 Issue/Ticket** — Jira / GitHub Issue
- **PR テンプレート** — `.github/pull_request_template.md` があれば優先採用
- **対象ブランチ** — main / develop / release/* 等

## 構成(要点)

PR は以下のセクションを含む Markdown で生成する。具体的なテンプレートは PR 種別ごとに [references/templates.md](references/templates.md) を参照(汎用 / Bugfix / Refactor / Infra)。

主要セクション:

- **概要 / Summary** — 何を、なぜ変更したか(1-2文)
- **背景 / Context** — 動機、関連 Issue・ドキュメント
- **変更内容 / Changes** — 機能・改修・テスト追加の箇条書き
- **動作確認 / How to Test** — レビュアーが手元で再現できる手順
- **影響範囲 / Impact** — ユーザー / DB / 環境変数 / API 互換性
- **レビュー観点 / Review Notes** — 重点的に見てほしい箇所
- **チェックリスト** — テスト・ドキュメント・CHANGELOG 等

## 種別別の重点

| PR 種別 | 重点的に書くべき項目 |
| :--- | :--- |
| 機能追加 | 動機、ユーザー影響、動作確認手順、互換性 |
| Bugfix | 再現手順、原因、修正方針、回帰テスト |
| Refactor | 挙動非変更の証拠(テスト、カバレッジ)、なぜ今やるか |
| Infra | ロールバック手順、監視項目、ステージング検証 |

## 原則

- **「なぜ」を最優先で書く。** 「何を変えたか」は diff から読める。動機・意図・選択しなかった代替案を書く。
- **レビュアー目線で書く。** どこを重点的に見るべきか、どの順序で読むと理解しやすいかを案内する。
- **動作確認手順は再現可能に。** 「動きました」だけでなく、レビュアーが手元で追える手順。
- **影響範囲を正直に書く。** 「影響なし」と書くなら、その根拠もセットで。
- **既存テンプレートを尊重。** `.github/pull_request_template.md` があればそのフォーマットを採用。

## 関連スキル

- `commit-msg` — このコミット群を作成するとき
- `changelog-writer` — リリース時に PR をユーザー視点で集約
