---
name: api-doc
description: RESTのルート定義からOpenAPI仕様書のドラフトを生成する。API仕様書・Swagger・エンドポイント定義のまとめ依頼で使う。
---

# API Documentation

実装コードから OpenAPI 3.1 仕様書ドラフトを生成する。

## 抽出対象

ルート定義から以下を読み取る:

1. **パス・メソッド** — `GET /users/:id` 等
2. **パスパラメータ** — `:id` の型・必須性
3. **クエリパラメータ** — 名前・型・必須性・デフォルト値
4. **リクエストボディ** — DTO/スキーマ定義から逆引き
5. **レスポンスボディ** — 戻り値型、エラー時のシェイプ
6. **ステータスコード** — 明示的に返す `200/201/204/400/401/403/404/409/422/500` 等
7. **認証** — ミドルウェア/デコレータから判定(`@Authenticated`, `requireAuth` 等)
8. **タグ** — コントローラ名/ファイル名からグルーピング

## フレームワーク別 検出ポイント

| フレームワーク | 検出箇所 |
| :--- | :--- |
| Express | `app.get('/path', ...)`, Joi/Zodバリデータ |
| Fastify | `fastify.route({ schema: ... })` |
| NestJS | `@Controller`, `@Get/@Post`, DTOクラス, `@ApiProperty` |
| FastAPI | パスオペレータ関数 + Pydantic モデル |
| Spring | `@RestController`, `@RequestMapping`, `@RequestBody` |
| Rails | `routes.rb`, ストロングパラメータ |

## 出力フォーマット

OpenAPI 3.1 YAML を生成する。フル例は [references/openapi-example.md](references/openapi-example.md) を参照。
構造の要点:

- `info` / `servers` / `tags` をヘッダに置く
- `paths` に各エンドポイント、`parameters` / `responses` / `security` を明示
- `components.schemas` で再利用される型を定義(`$ref` 参照)
- 共通レスポンス(Unauthorized 等)も `components.responses` に集約

## 原則

- **「ドラフト」と明記する。** 自動生成は不完全。レビューを前提とする。
- **examples を最低 1 つ付ける。** 利用者にとって schema 単独より具体例が圧倒的に有益。
- **エラーレスポンスを統一化する。** プロジェクト共通の `Error` schema があれば全エンドポイントで参照する。
- **既存の OpenAPI があれば追記。** 全文書き換えではなくマージ。
- **`additionalProperties: false` をデフォルト推奨。** 想定外プロパティを許容しないほうが堅牢。
- **読み取れない情報は推測せず TODO コメント。** pagination 形式・認証方式等が不明なら明示する。
