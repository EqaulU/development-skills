---
name: a11y-review
description: アクセシビリティ(WCAG 2.1/2.2のAA基準)観点でUIコードをレビューする。セマンティックHTML、ARIA属性、コントラスト比、キーボード操作、スクリーンリーダー対応をチェック。a11y・アクセシビリティ・WCAG・スクリーンリーダー対応の依頼で使う。
---

# Accessibility Review

UI コードのアクセシビリティを WCAG 2.1/2.2 AA 基準でレビューする。

## チェック観点

### セマンティック HTML
- `div`/`span` で済ませているクリック可能要素 → `button` / `a` に
- 見出しが正しい階層(h1 → h2 → h3、スキップしない)
- ランドマーク要素(`header`/`nav`/`main`/`aside`/`footer`)の使用
- リストは `ul`/`ol`/`li`、表は `table`/`th`/`scope`

### ARIA
- ネイティブで表現できるものに ARIA を追加しない(`role="button"` on `<button>` は不要)
- 状態は ARIA で表現(`aria-expanded`, `aria-selected`, `aria-pressed`, `aria-busy`)
- フォーム要素には `label`(`for`+`id` または包含)。`aria-label` は最後の手段
- 動的更新は `aria-live`(polite/assertive)で通知

### キーボード操作
- すべての対話要素が `Tab` で到達でき、`Enter`/`Space` で操作可能
- フォーカス可視(`:focus-visible` でアウトラインを残す、`outline: none` のみは NG)
- フォーカストラップ(モーダル開閉時にダイアログ外にフォーカスが逃げない)
- カスタムコンポーネント(combobox / tabs / menu)は WAI-ARIA Authoring Practices に従う

### コントラスト
- 通常テキスト 4.5:1、大きいテキスト/UI 要素 3:1 以上
- 色のみで情報を伝えない(エラーは色+アイコン+テキスト)
- ホバー/フォーカス時のコントラストも確保

### 画像・メディア
- `<img>` には `alt`(装飾なら `alt=""`)
- 動画には字幕(`<track kind="captions">`)、音声には書き起こし
- アイコンボタンには可視ラベルか `aria-label`

### フォーム
- エラーメッセージは入力欄と紐付け(`aria-describedby`)
- 必須項目は `aria-required` + 視覚表示
- 入力種別に合わせた `type`(`email`, `tel`, `url`, `number`)
- `autocomplete` 属性の適切な指定

## 出力フォーマット

```markdown
## アクセシビリティ診断結果

### 🔴 Critical(WCAG AA 違反)
- **[WCAG 1.4.3 コントラスト]** ボタンのテキスト色 #888 が背景 #fff に対して 3.5:1
  - 該当: `Button.tsx:45`
  - 修正: `#666` 以上の濃度に
- **[WCAG 2.1.1 キーボード操作]** カスタムドロップダウンが Tab で到達できない
  - 該当: `Dropdown.tsx:80`
  - 修正: `tabindex="0"` + Enter/Space ハンドラ

### 🟡 改善推奨
- **[セマンティック]** `<div onClick>` を `<button>` に
- **[ARIA]** モーダル開閉時のフォーカス管理

### ✅ 確認済み
- 主要画面のコントラスト
- フォームの label 関連付け

### 推奨ツール
- `axe-core` / `@axe-core/playwright` での自動チェック
- VoiceOver / NVDA での実機確認
```

## 原則

- **ネイティブ HTML 要素を最優先。** ARIA は補助。ネイティブで表現できるものを ARIA で再発明しない。
- **自動テストは網羅性 60% 程度。** axe-core 等で機械的に検出できるのは一部。実機/スクリーンリーダーでの確認が必須。
- **「動く」だけでなく「使える」を目指す。** Tab で到達できても、フォーカス順序が直感的でなければ失敗。
- **国際標準に従う。** WAI-ARIA Authoring Practices のパターンを真似る。独自実装を作らない。
- **障害特性を意識する。** 視覚障害だけでなく、運動障害(キーボード)、認知障害(明確な言葉遣い)、聴覚障害(字幕)も対象。

## 関連スキル

- `code-review` — UI コード一般のレビュー(a11y は重点観点として組み込む)
- `e2e-scenario` — キーボード操作・スクリーンリーダーのシナリオも組み込む
