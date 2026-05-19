# OpenAPI 3.1 出力例

`api-doc` skill が生成する OpenAPI 仕様書のリファレンス例。
実際の出力時はこの構造に倣う。

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
