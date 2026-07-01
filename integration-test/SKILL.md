---
name: integration-test
description: コンポーネント境界をまたぐ結合テスト(API↔サービス↔実DB、DAO↔DB、外部APIスタブ、トランザクション・フィクスチャ設計)を生成する。結合テスト・インテグレーションテスト・IT追加の依頼で使う。
---

# Integration Test

ユニットがモックで消している「継ぎ目」を、本物の依存で検証する結合テストを生成する。

## いつ使うか

- API → サービス → DB のように、複数コンポーネントを通した動作を確かめたいとき
- リポジトリ/DAO の SQL・トランザクション・制約が実DBで正しく効くか確認したいとき
- 自コードと外部API/キューの結線を、E2E より速く・原因特定しやすく検証したいとき

`test-gen` は単一の関数・クラス(依存はモック)。`e2e-scenario` はユーザーストーリー全体をスタック越しに。`integration-test` はその中間で、**コンポーネント境界が実際に噛み合うか**を受け持つ。

## 入力情報

開始時に把握する(不明なら質問):

- **対象の結合点** — どことどこの境界か(API↔DB、サービス間、外部API 連携 等)
- **使えるテスト基盤** — テストDB(Testcontainers / インメモリ / 共有)、CI の制約
- **外部依存** — 課金・メール・サードパーティAPI・キューの有無
- **既存の結合テスト** — あれば命名・ハーネス・独立性の取り方を踏襲

## 何を本物・何をスタブにするか

結合テストの肝は「どこまで本物を通すか」。既定方針:

| 依存 | 既定 | 理由 |
| :--- | :--- | :--- |
| DB | **本物**(テストDB) | SQL・制約・トランザクション・マイグレーションの正しさが検証対象そのもの |
| 同プロセスの自社モジュール/サービス | **本物** | 結線(配線・DI・シリアライズ)の検証が目的 |
| 外部API(課金・メール・サードパーティ) | **スタブ** | 不安定・課金・レート制限。契約をスタブで固定する |
| メッセージキュー | **本物**(テスト用)or インメモリ | publish/consume の結線を検証 |
| 時刻・乱数・UUID | **固定/注入** | 再現性のため |

「全部本物」は遅く脆く、「全部モック」はユニットと変わらない。**検証したい継ぎ目だけ本物**にする。

## よく検証する結合点

1. **HTTP API → アプリ層 → 実DB** — リクエストを投げ、レスポンスと**DB状態の副作用**を両方検証
2. **リポジトリ/DAO ↔ 実DB** — SQL の正しさ、制約違反、トランザクション境界、マイグレーション互換
3. **サービス間の結線** — 配線・DI・設定が実際に組み上がるか
4. **自コード ↔ 外部API** — スタブサーバで契約(リクエスト形・エラー・タイムアウト)を固定
5. **キュー/イベントの produce → consume** — 発行したメッセージが意図通り処理されるか

## テストDBと独立性(要点)

- **起動**: 本番同等イメージの Testcontainers が第一候補。軽さ優先ならインメモリ(SQLite 等、ただし方言差に注意)
- **独立性**: 各テストをトランザクションで包んでロールバック、または毎回 truncate。**テスト間で状態を共有しない**
- **スキーマ**: 本番と同じマイグレーションを適用する(手書きの DDL でズレを作らない)

フレームワーク別のハーネス設定(Testcontainers 起動、トランザクション巻き戻し、外部APIスタブの定番ツール、CI・flaky 対策)は [references/harness-setup.md](references/harness-setup.md) を参照。

## 出力フォーマット(例: TypeScript + Vitest + supertest + 実テストDB)

```typescript
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import request from 'supertest';
import { app } from '../src/app';
import { db, migrate, truncateAll } from './helpers/testDb';

// 結合テスト: HTTP → サービス → 実DB を一気通貫で通す
describe('POST /v1/orders (integration)', () => {
  beforeAll(async () => {
    await migrate(db);            // 本番と同じマイグレーションをテストDBに適用
  });
  beforeEach(async () => {
    await truncateAll(db);        // テスト間の独立性: 毎回クリーンな状態に
    await db.insert('products', { sku: 'PC-1', stock: 1, price: 1000 });
  });
  afterAll(async () => {
    await db.close();
  });

  it('在庫がある商品を注文すると201で永続化され、在庫が減る', async () => {
    // Act: 実エンドポイントを叩く
    const res = await request(app)
      .post('/v1/orders')
      .set('Idempotency-Key', 'key-1')
      .send({ items: [{ sku: 'PC-1', quantity: 1 }] });

    // Assert: レスポンス
    expect(res.status).toBe(201);
    expect(res.body.status).toBe('pending');

    // Assert: DB状態 — この副作用検証がユニットでは見えない継ぎ目
    const order = await db.queryOne('SELECT * FROM orders WHERE id = $1', [res.body.id]);
    expect(order).not.toBeNull();
    const product = await db.queryOne('SELECT stock FROM products WHERE sku = $1', ['PC-1']);
    expect(product.stock).toBe(0);
  });

  it('同じIdempotency-Keyの再送は二重作成しない', async () => {
    const send = () => request(app).post('/v1/orders')
      .set('Idempotency-Key', 'key-2')
      .send({ items: [{ sku: 'PC-1', quantity: 1 }] });
    const first = await send();
    const second = await send();

    expect(second.body.id).toBe(first.body.id);
    const count = await db.queryOne('SELECT count(*)::int AS c FROM orders');
    expect(count.c).toBe(1);      // 2件目が作られていないことを実DBで確認
  });

  it('在庫切れなら409を返し、注文は作られない(トランザクションの整合)', async () => {
    await db.update('products', { stock: 0 }, { sku: 'PC-1' });
    const res = await request(app).post('/v1/orders')
      .set('Idempotency-Key', 'key-3')
      .send({ items: [{ sku: 'PC-1', quantity: 1 }] });

    expect(res.status).toBe(409);
    const count = await db.queryOne('SELECT count(*)::int AS c FROM orders');
    expect(count.c).toBe(0);
  });
});
```

## 原則

- **検証したい継ぎ目だけ本物にする。** 全部本物は遅く脆く、全部モックはユニットと同じ。目的の境界に絞る。
- **副作用を必ず検証する。** レスポンスだけでなく、DB 状態・送信メッセージ・外部呼び出し回数など「向こう側」を確認する。
- **テストは独立に。** 実行順序や前テストの残骸に依存させない。毎回クリーンな状態から始める(ロールバック/truncate)。
- **スキーマは本番マイグレーションで作る。** テスト用に DDL を手書きすると、本番とのズレを検証できなくなる。
- **外部APIはスタブで契約を固定。** 本物を叩くと不安定・課金・レート制限。リクエスト形とエラー応答をスタブで再現する。
- **flaky を放置しない。** 共有状態・時刻依存・非同期待ちの不備が主因。`references/harness-setup.md` の対策に沿って潰す。

## 関連スキル

- `test-gen` — その下層。関数・クラス単位のユニット(依存はモック)
- `e2e-scenario` — その上層。ユーザーストーリー全体をスタック越しに検証
- `edge-case-finder` — 結合点で起こりうる境界・異常パターンの洗い出し
- `data-model` / `api-design` — 設計したスキーマ・API契約が実際に噛み合うかを結合テストで裏取り
- `migration-plan` — マイグレーション適用後も結合テストが通るかで互換性を担保
