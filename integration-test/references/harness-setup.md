# 結合テスト ハーネス設定リファレンス

`integration-test` skill 本体から呼び出される、テスト基盤の組み方の詳細。
テストDBの用意・独立性の確保・外部依存のスタブ・言語別の定番・flaky 対策・CI をまとめる。

## テストDBの用意

| 方式 | 仕組み | 利点 | 注意 |
| :--- | :--- | :--- | :--- |
| Testcontainers | 本番同等の DB をコンテナで起動 | 方言差なし、本番に最も近い | Docker 必須、起動コスト |
| インメモリ(SQLite 等) | プロセス内 DB | 速い、依存が軽い | SQL 方言・型・制約が本番と乖離しうる |
| 共有テストDB | 常設の専用 DB に接続 | 起動不要 | 並列・他テストとの干渉対策が必須 |

**原則**: 検証対象が「実 DB の挙動」なら Testcontainers で本番同等イメージを使う。速度優先でインメモリにする場合は、方言依存の SQL を別途本番相当 DB でも回す。スキーマは**本番と同じマイグレーション**を適用して作る(手書き DDL でズレを作らない)。

## テスト間の独立性

状態がテスト間で漏れると flaky の温床になる。主な戦略:

| 戦略 | やり方 | 向き/注意 |
| :--- | :--- | :--- |
| トランザクション巻き戻し | 各テストを BEGIN し、終了時に ROLLBACK | 速い。ただしコードが自前で commit する/別コネクションを使うと効かない |
| truncate / delete | 各テスト前に対象テーブルを空に | 確実。FK 順序や `TRUNCATE ... CASCADE`、`RESTART IDENTITY` に注意 |
| テストごとにスキーマ/DB | 並列ワーカーごとに分離 | 並列実行に強い。準備コスト高め |

- シードは「各テストが必要な最小データ」を `beforeEach` で投入。グローバルな共有シードに依存しない
- 採番(serial/identity)に依存したアサーションを避ける(`RESTART IDENTITY` か、ID は返り値から取得)

## 外部依存のスタブ

外部 HTTP API は本物を叩かない(不安定・課金・レート制限)。リクエスト形とエラー応答を固定する。

| 言語/領域 | 定番ツール |
| :--- | :--- |
| Node.js | `nock`、`msw`(Service Worker/Node)、`@mswjs/interceptors` |
| Python | `responses`、`respx`(httpx)、`vcrpy`(記録再生) |
| Java/JVM | WireMock、MockServer |
| Ruby | WebMock、VCR |
| 言語非依存 | WireMock / MockServer をコンテナ起動して実 HTTP で差し込む |

- **契約テスト**: 提供側/消費側の食い違いを防ぐなら Pact 等のコントラクトテストを併用
- スタブは**正常応答だけでなく**、タイムアウト・5xx・不正ボディも用意してエラー処理を結合検証する
- メール・SMS・決済は専用のサンドボックス/スタブ(MailHog、決済プロバイダの test mode)を使う

## メッセージキュー/イベント

- テスト用ブローカ(Testcontainers の Kafka/RabbitMQ 等)か、インメモリ実装で publish→consume を検証
- 非同期は「ポーリングで条件成立を待つ」(固定 sleep は flaky)。タイムアウト付きの待機ヘルパを用意

## 言語/フレームワーク別の定番

### Node.js(Vitest/Jest)
- HTTP: `supertest` でアプリインスタンスに直接リクエスト
- DB: Testcontainers(`@testcontainers/postgresql` 等)or テスト用 Docker Compose
- 外部API: `nock` / `msw`
- 独立性: `beforeEach` で truncate、または ORM のトランザクションラッパ

### Python(pytest)
- HTTP: FastAPI `TestClient` / Flask `test_client` / Django `Client`
- DB: `testcontainers`、`pytest-postgresql`、または `--reuse-db`。フィクスチャでトランザクションをロールバック
- 外部API: `responses` / `respx`
- 独立性: `@pytest.fixture` でトランザクション境界を管理(`django.test.TransactionTestCase` 等)

### Java(Spring Boot)
- `@SpringBootTest` + `MockMvc`/`WebTestClient`、`@Testcontainers` + `@DynamicPropertySource` で接続情報注入
- DB: Testcontainers(PostgreSQLContainer 等)。`@Transactional` テストで自動ロールバック
- 外部API: WireMock(`@AutoConfigureWireMock` 等)

### Ruby(RSpec)
- `rack-test` / request spec
- DB: `database_cleaner`(transaction/truncation 戦略)
- 外部API: WebMock / VCR

### Go
- `net/http/httptest` でハンドラを起動、`dockertest` or Testcontainers-Go で DB
- 外部API: `httptest.Server` でスタブ

## flaky(不安定テスト)の主因と対策

| 主因 | 対策 |
| :--- | :--- |
| テスト間の状態共有 | 独立性戦略を徹底(ロールバック/truncate)、グローバル可変状態を排除 |
| 時刻・タイムゾーン依存 | 時刻を注入/固定(クロック抽象化)、UTC で統一 |
| 非同期の固定 sleep | 条件ポーリング + タイムアウト待機に置換 |
| 実行順序依存 | ランダム順で流して検出。順序前提のテストを直す |
| 並列実行の競合 | ワーカーごとに DB/スキーマ分離、ユニークなテストデータ |
| 外部ネットワーク | 必ずスタブ化。本物の外部接続を結合テストに残さない |

## CI 上の考慮

- Docker(Testcontainers)が使える CI ランナーか確認。使えないなら共有テストDB + 分離戦略
- ユニットより遅いので、別ジョブ/別ステージに分けて並列化
- 起動コストの高いコンテナはスイート単位で再利用(テストごとに起動しない)
- 失敗時のログ(クエリ・スタブのマッチ状況)を出力して原因追跡を容易に
