---
name: readme-update
description: コード変更・新機能・依存変更に合わせてREADMEを更新する(インストール手順・使用例・設定項目の同期)。README更新の依頼で使う。
---

# README Update

コード変更に合わせてREADMEを更新する。差分から「READMEに反映すべき変更」を抽出し、必要箇所だけ更新する。

## 更新フロー

1. **現在のREADMEを読む** — 既存の構成、トーン、言語(日本語/英語)、見出しレベルを把握
2. **変更内容を把握** — 直近のコミット、PR、diff、または明示的に伝えられた変更点
3. **影響セクションを特定** — 下表で対応
4. **必要箇所のみ更新** — 全文書き換えは避ける。差分が分かるように

## 変更内容 → 更新箇所マッピング

| 変更内容 | 更新箇所 |
| :--- | :--- |
| 新機能追加 | Features / Usage / Examples |
| 新コマンド/API追加 | Usage / API Reference |
| 依存パッケージ追加 | Installation / Requirements |
| 設定項目追加 | Configuration / Environment Variables |
| 環境要件変更(Node 18→20等) | Requirements / Prerequisites |
| 破壊的変更 | Migration Guide / Breaking Changes |
| ファイル構成変更 | Project Structure |
| デプロイ方法変更 | Deployment |
| Contributing方針変更 | Contributing |

## README構成テンプレート(新規作成時)

```markdown
# プロジェクト名

1〜2行の説明(何ができるか、なぜ作ったか)

[![CI](badge.svg)] [![License](badge.svg)]

## 特徴
- 主要機能を箇条書きで3-5個

## クイックスタート
最小限で動かす手順(コピペで動くコマンド)

## インストール
\`\`\`bash
npm install
\`\`\`

## 使い方
最も典型的なユースケース1つ

## 設定
| 環境変数 | 必須 | デフォルト | 説明 |
| :--- | :--- | :--- | :--- |
| DATABASE_URL | yes | - | DB接続文字列 |

## API / コマンド
主要なエントリーポイント

## 開発
\`\`\`bash
npm run dev
npm test
\`\`\`

## 貢献
CONTRIBUTING.md 参照

## ライセンス
MIT
```

## 原則

- **既存トーン・言語を保つ。** 日本語READMEに英語セクションを足さない(逆も同様)。
- **差分が小さくなるように。** 不要な再構成はしない。レビュアーが変更点を追えるように。
- **コピペで動くコマンドを書く。** 省略記号 `...` や疑似コマンドは避ける。
- **スクリーンショット/GIFは内容変更時のみ。** 古いUIのまま残らないように、変更があれば差し替えを促す。
- **目次は長いREADMEのみ。** 短いREADMEに目次は不要。
- **「クイックスタート」は守る。** 5分以内に動かせる構成を維持。
