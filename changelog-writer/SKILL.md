---
name: changelog-writer
description: コミット履歴・PRからCHANGELOG.mdやリリースノートを生成する。Keep a Changelog形式、SemVer判定を含む。リリース前の変更履歴まとめ依頼で使う。
---

# Changelog Writer

コミット履歴やPRリストから CHANGELOG.md を生成・更新する。

## 入力情報の確認

開始前に以下を把握する:

1. **比較範囲** — `git log v1.2.0..HEAD` または PR一覧 (タグが無ければユーザーに範囲確認)
2. **既存CHANGELOGの形式** — Keep a Changelog / 独自形式 / 無し
3. **コミット規約** — Conventional Commits を採用しているか
4. **対象バージョン** — 新バージョン番号(自動算出可能なら提案)

## コミット収集

```bash
# 前回タグ以降のコミット
git log --no-merges --pretty=format:"%h %s" $(git describe --tags --abbrev=0)..HEAD

# PRベースの場合
git log --merges --pretty=format:"%h %s" v1.2.0..HEAD
```

## Conventional Commits → カテゴリマッピング

| コミット接頭辞 | CHANGELOGカテゴリ | SemVer影響 |
| :--- | :--- | :--- |
| `feat:` | Added | minor |
| `fix:` | Fixed | patch |
| `perf:` | Changed (perf) | patch |
| `refactor:` | Changed (internal) | patch |
| `docs:` | (通常省略) | none |
| `test:` | (通常省略) | none |
| `chore:` | (通常省略) | none |
| `feat!:` / `BREAKING CHANGE:` | Changed (Breaking) | major |
| `revert:` | Removed / Reverted | patch |
| `deprecate:` | Deprecated | minor |
| `remove:` | Removed | major |
| `security:` | Security | patch |

## 出力フォーマット(Keep a Changelog)

```markdown
# Changelog

すべての変更はこのファイルに記録される。
形式は [Keep a Changelog](https://keepachangelog.com/) に準拠し、[Semantic Versioning](https://semver.org/) を採用している。

## [Unreleased]

## [1.3.0] - 2026-05-17

### Added
- ユーザープロフィール画像のアップロード機能 ([#123](link))
- ダークモード対応 ([#125](link))

### Changed
- 検索結果の表示順を更新日時降順に変更 ([#127](link))

### Deprecated
- `GET /api/v1/users/me` は v2.0 で削除予定。代わりに `/api/v2/me` を使用 ([#130](link))

### Fixed
- 大量データインポート時のメモリリーク ([#132](link))
- iOS Safari でログインボタンが反応しない問題 ([#134](link))

### Security
- 依存パッケージ `xxx@1.2.3` の脆弱性 CVE-2026-XXXXX 対応 ([#136](link))

### Removed
- 廃止済みエンドポイント `/api/v0/*` を削除 ([#140](link))

[Unreleased]: https://github.com/org/repo/compare/v1.3.0...HEAD
[1.3.0]: https://github.com/org/repo/compare/v1.2.0...v1.3.0
```

## コミット文 → ユーザー視点への書き換え

開発者向けの簡潔なコミットメッセージを、READMEを読むユーザー視点に翻訳する:

| コミット | CHANGELOG |
| :--- | :--- |
| `fix: null check in UserService.findById` | ユーザー検索で稀に発生していたエラーを修正 |
| `feat: add Redis cache layer` | レスポンス速度を改善(キャッシュ層を導入) |
| `refactor: extract validation logic` | (内部リファクタリング、ユーザー影響なし → 省略 or "内部改善") |

## 原則

- **ユーザー視点で書く。** 「`UserController.list` を修正」ではなく「ユーザー一覧取得が速くなった」と書く。
- **PR番号やIssue番号をリンク化。** 詳細を追える状態を保つ。
- **`chore` `docs` `test` は基本省略。** ユーザー価値が無いものは載せない。例外はインフラ要件の変更など。
- **Breaking Changesは目立たせる。** 専用セクションかつ移行手順への導線を必ず付ける。
- **SemVerを提案。** `feat!` あれば major、`feat` あれば minor、それ以外は patch を推奨。
- **過去エントリは触らない。** 既存の `[1.2.0]` セクションを再編集しない。
