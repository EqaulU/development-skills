---
name: e2e-scenario
description: ユーザーストーリーからE2Eシナリオ(Gherkin/Given-When-Then)を作成する。Happy/Alternative/Error Pathを網羅。受け入れテスト・BDD作成の依頼で使う。
---

# E2E Scenario Generation

ユーザーストーリーや機能要件から、E2Eテストシナリオ(受け入れテスト)を生成する。

## 入力情報の確認

シナリオ生成前に以下を把握する。不足していれば質問する。

1. **対象機能** — 何の機能か、誰が使うか
2. **ユーザーストーリー** — "As a [役割], I want [目的], so that [価値]"
3. **テスト対象レイヤー** — UI(Playwright/Cypress)/ API(REST/GraphQL)/ 両方
4. **前提データ** — テスト前に必要な状態(ユーザー登録済み、商品マスター存在等)

## シナリオ設計

各ストーリーに対して以下3種を最低1つずつ作る:

- **Happy Path** — 期待通りに進む典型シナリオ
- **Alternative Path** — 正しいが分岐する経路(例: 既存ユーザーログイン vs 新規登録)
- **Error Path** — 失敗・エラー処理(バリデーション、権限エラー、外部障害)

## 出力フォーマット(Gherkin形式)

```gherkin
Feature: ECサイトでの商品購入
  As a 登録済みユーザー
  I want 商品をカートに入れて決済する
  So that 商品を購入できる

  Background:
    Given ユーザー "yamada@example.com" が登録済みである
    And 商品 "ノートPC" が在庫1台で登録されている

  Scenario: Happy Path - 通常購入
    Given "yamada@example.com" でログイン済み
    When 商品 "ノートPC" をカートに追加する
    And チェックアウト画面で支払い方法 "クレカ" を選択する
    And 注文を確定する
    Then 注文完了画面が表示される
    And 注文確認メールが "yamada@example.com" に送信される
    And 在庫が 0 になる

  Scenario: Error Path - 在庫切れ
    Given 商品 "ノートPC" の在庫が 0 である
    When カート画面を開く
    Then "在庫切れ" のメッセージが表示される
    And 注文ボタンが無効化されている

  Scenario Outline: バリデーション
    When 数量に "<input>" を入力する
    Then "<message>" が表示される

    Examples:
      | input | message |
      | 0     | 1以上を入力してください |
      | -1    | 1以上を入力してください |
      | 999   | 在庫数を超えています |
```

## ツール固有変換

Gherkinから各ツールに落とすときの参考:
- **Playwright/Cypress** — `test.describe` / `it.each` で表現
- **Postman/Newman** — Collection の Request + Test スクリプト
- **手動テスト用** — Markdownのチェックリスト形式

## 原則

- **ビジネス言語で書く。** 実装詳細(クラス名・SQL)はシナリオに混ぜない。読み手は非開発者も含む。
- **データ依存を明示する。** Background/Givenで前提を必ず宣言。テストの再現性が大きく変わる。
- **アサーションは複数観点で。** 画面表示だけでなく、DB状態・送信メール・ログ等の副作用も検証。
- **依存を最小化する。** あるシナリオが別のシナリオの実行結果に依存しないように。
