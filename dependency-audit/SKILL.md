---
name: dependency-audit
description: 依存パッケージのCVE・ライセンス・メンテ状況・サイズ影響をスキャンする。npm audit等の結果解釈、依存更新・ライセンスチェックの依頼で使う。
---

# Dependency Audit

依存パッケージのリスクと健全性を確認する。

## チェック観点

### 1. セキュリティ脆弱性
- CVE が報告されているパッケージ
- 脆弱性の深刻度(Critical / High / Medium / Low)
- 影響範囲(直接依存 / 推移依存)
- 修正版が出ているか / ワークアラウンドはあるか

### 2. ライセンス
- 商用利用可否(MIT / Apache-2.0 / BSD / ISC は通常OK)
- コピーレフト(GPL / AGPL / LGPL)が混入していないか
- ライセンス不明 / カスタムライセンスの存在
- 帰属表示の必要性

### 3. メンテナンス状況
- 最終リリース日(2年以上更新なし → 要注意)
- メンテナの数、活動度
- オープンIssue数 vs クローズ数
- スター数の急減・急増

### 4. サイズ・パフォーマンス影響
- bundle size(フロントエンド) / install size(バックエンド)
- 同じ目的の軽量代替品があるか
- Tree-shaking が効くか

### 5. 重複・整理
- 同じ機能のパッケージが複数(`moment` と `date-fns` の併用等)
- 推移依存の重複バージョン
- 未使用の依存(`depcheck` 等で検出)

## ツール別コマンド

```bash
# Node.js
npm audit
npm audit --json | jq                       # 機械処理用
npm outdated
npx depcheck                                # 未使用依存
npx license-checker --summary               # ライセンス一覧

# Python
pip-audit
pip list --outdated
pip-licenses

# Ruby
bundle audit check --update
bundle outdated

# Rust
cargo audit
cargo outdated

# Go
govulncheck ./...

# 汎用(GitHub)
# Dependabot alerts を確認
```

## 出力フォーマット

```markdown
## 依存パッケージ監査結果

### 🚨 Critical / High 脆弱性
| パッケージ | バージョン | CVE | 深刻度 | 修正版 | 対応 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| lodash | 4.17.20 | CVE-2021-23337 | High | 4.17.21 | `npm update lodash` で解決 |

### ⚠️ Medium 脆弱性
(同形式)

### 📜 ライセンス懸念
- `xxx@1.2.3` — GPL-3.0 を検出。商用プロダクトでの使用は要確認

### 🪦 メンテナンスリスク
- `legacy-pkg@0.5.0` — 最終更新 3年前、Issue 100件以上未対応
  - 代替: `modern-alt` を検討

### 📦 サイズ影響
- `moment` (290KB) → `date-fns` (50KB) or `dayjs` (10KB) への移行で約 240KB 削減可能

### 🧹 整理候補
- 未使用: `unused-pkg` (depcheck 検出)
- 重複機能: `axios` と `node-fetch` の併用 → 統一を推奨

### 推奨アクション(優先順)
1. Critical/High の即時パッチ適用(影響範囲確認後)
2. Medium の計画的アップデート
3. GPL ライセンスの代替検討
4. 未使用依存の削除
```

## 原則

- **`npm audit fix --force` を盲信しない。** メジャーバージョン上げを伴うことがあり、破壊的変更が含まれる。
- **推移依存の脆弱性は overrides / resolutions を検討。** 親パッケージが更新を取り込むまで待てない場合に。
- **「自分のコードが脆弱パスを通っているか」も考慮。** 報告されていても、未使用の関数なら実害は低い。優先度判断材料に。
- **アップデートは段階的に。** 全部一気に上げるとどれが原因で壊れたか分からなくなる。
- **自動化を勧める。** Dependabot / Renovate でPRが自動で来る運用を推奨。
- **`postinstall` スクリプトに注意。** サプライチェーン攻撃の典型経路。新規依存追加時は `npm pack` で中身を確認する習慣を。
