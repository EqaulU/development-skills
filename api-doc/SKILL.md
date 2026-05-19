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

## 出力フォーマット(OpenAPI 3.1 YAML)

```yaml
openapi: 3.1.0
info:
  title: My Service API
  version: 1.0.0
  description: |
    自動生成ドラフト。レビュー後に確定してください。

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: http://localhost:3000/v1
    description: Local

tags:
  - name: users
    description: ユーザー管理

paths:
  /users/{id}:
    get:
      summary: ユーザー1件取得
      tags: [users]
      security:
        - bearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: ユーザーが存在しない
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '401':
          $ref: '#/components/responses/Unauthorized'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    User:
      type: object
      required: [id, email, createdAt]
      properties:
        id: { type: string, format: uuid }
        email: { type: string, format: email }
        name: { type: string, nullable: true }
        createdAt: { type: string, format: date-time }

    Error:
      type: object
      required: [code, message]
      properties:
        code: { type: string }
        message: { type: string }
        details: { type: object, additionalProperties: true }

  responses:
    Unauthorized:
      description: 認証エラー
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
```

## 不確実性の扱い

実装から読み取れない情報は推測せず、コメントで明示:

```yaml
# TODO: レスポンスのpagination形式を確認(Cursor or Offset?)
```

## 原則

- **「ドラフト」と明記する。** 自動生成は不完全。レビューを前提とする。
- **examples を最低1つ付ける。** 利用者にとってschema単独より具体例が圧倒的に有益。
- **エラーレスポンスを統一化する。** プロジェクト共通の `Error` schema があれば全エンドポイントで参照する。
- **既存のOpenAPIがあれば追記。** 全文書き換えではなくマージ。
- **`additionalProperties: false` をデフォルト推奨。** 想定外プロパティを許容しないほうが堅牢。
