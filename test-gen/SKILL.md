---
name: test-gen
description: 関数・クラスのユニットテストを生成する(Jest/Vitest/pytest/JUnit/RSpec等、AAAパターン、正常/異常/境界値網羅)。テスト追加・カバレッジ向上の依頼で使う。
---

# Test Generation

対象コードからユニットテストを生成する。

## 生成フロー

1. **対象を読み解く** — 関数シグネチャ、入力ドメイン、副作用、依存(DB/API/時刻/乱数)を把握
2. **テストケースを設計** — 正常系・異常系・境界値・例外を網羅(`edge-case-finder` skillがあれば併用)
3. **フレームワークを特定** — package.json / requirements / Gemfile / pom.xml から既存のテストランナーを確認
4. **既存テストのスタイルを踏襲** — 既にテストがあれば、命名・構造・モック方法を合わせる
5. **AAAパターンで書く** — Arrange(準備) → Act(実行) → Assert(検証) を明確に分ける
6. **モックは最小限** — 本物で動かせる依存は本物で。外部I/O・時刻・乱数だけモック

## テストケース設計の網羅指針

| 種別 | 例 |
| :--- | :--- |
| 正常系 | 期待される典型入力 |
| 境界値 | 0, 1, 最大値, 空配列, 1要素配列 |
| 異常系 | null/undefined, 不正型, 範囲外 |
| 例外パス | throwされるべき条件で実際にthrowするか |
| 副作用 | DB書き込み、外部API呼び出し回数の検証 |
| 冪等性/並行性 | 必要なら複数回呼び出し、同時実行 |

## 出力フォーマット(例: TypeScript + Vitest)

```typescript
import { describe, it, expect, vi } from 'vitest';
import { calculateDiscount } from './discount';

describe('calculateDiscount', () => {
  describe('正常系', () => {
    it('VIP会員には20%割引が適用される', () => {
      // Arrange
      const price = 1000;
      const member = { tier: 'vip' };
      
      // Act
      const result = calculateDiscount(price, member);
      
      // Assert
      expect(result).toBe(800);
    });
  });

  describe('境界値', () => {
    it('価格が0でもエラーにならない', () => { /* ... */ });
    it('価格が負の数の場合は0を返す', () => { /* ... */ });
  });

  describe('異常系', () => {
    it('memberがnullの場合は元の価格を返す', () => { /* ... */ });
  });
});
```

## 原則

- **テスト名は仕様書として読めるように。** 「〜のとき〜になる」の形式が望ましい。日本語OK。
- **1テスト1検証(基本)** — 1つの`it`で複数の概念を検証しない。失敗理由が曖昧になる。
- **テストデータは意味のある値に。** `"foo"` `123` より `"valid@example.com"` `userId: 42` のほうが意図が伝わる。
- **過剰なモックを避ける。** モックばかりだと実装変更でテストが壊れやすく、本当のバグを見逃す。
- **既存スタイルに合わせる。** 既存テストとの一貫性は理想形より優先。
