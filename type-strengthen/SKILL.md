---
name: type-strengthen
description: TypeScript型の強化(any/unknown排除、型ガード追加、Discriminated Union、ジェネリクス活用、satisfies演算子、Brand型、strict設定推奨)を行う。ユーザーが「型強化して」「any消して」「型安全にして」「型ガード書いて」「TypeScriptの型もっと厳しく」と言ったとき、`any` 多用や緩い型のコードを共有してきたとき、`/type-strengthen` と打ったときには必ずこのskillを使う。
---

# Type Strengthen (TypeScript)

緩い型を厳格化し、型システムにバグを発見させる。

## 段階的な強化レベル

| Lv | 内容 |
| :--- | :--- |
| 1 | `any` を `unknown` に置換 → 型ガードで絞り込み |
| 2 | 「string」「number」のような原始型を、意味のある型エイリアスや Branded Type で表現 |
| 3 | Discriminated Union で状態を表現(`{status: 'loading'} | {status: 'success', data: T}`) |
| 4 | `as` キャストを排除し、type predicate / assertion function に置き換え |
| 5 | ジェネリクスで重複型定義を統合 |
| 6 | `satisfies` で「型を満たしつつリテラル型を保つ」 |
| 7 | `tsconfig.json` の strict系オプション有効化 |

## よくある変換パターン

### any → unknown + 型ガード

```typescript
// Before
function handle(data: any) {
  return data.user.name;  // 何でも通る
}

// After
function isUserData(v: unknown): v is { user: { name: string } } {
  return typeof v === 'object' && v !== null
    && 'user' in v && typeof (v as any).user?.name === 'string';
}

function handle(data: unknown) {
  if (!isUserData(data)) throw new Error('invalid data');
  return data.user.name;  // 型安全
}
```

### Optional の連鎖 → Discriminated Union

```typescript
// Before: どのフィールドが入るかが暗黙
type Result = {
  isError?: boolean;
  data?: User;
  error?: string;
};

// After: 状態が明示的
type Result =
  | { kind: 'ok'; data: User }
  | { kind: 'error'; error: string };
```

### Branded Type で取り違え防止

```typescript
type UserId = string & { readonly __brand: 'UserId' };
type OrderId = string & { readonly __brand: 'OrderId' };

function asUserId(s: string): UserId { return s as UserId; }

// findUser(orderId) は型エラーになる
```

### satisfies で「定義の正しさ」と「リテラル保持」を両立

```typescript
// const に as const を付けるのに近いが、構造の型チェックも効く
const config = {
  retries: 3,
  mode: 'strict',
} satisfies { retries: number; mode: 'strict' | 'loose' };

// config.mode の型は 'strict'(リテラル)のまま
```

### tsconfig 推奨設定

```jsonc
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,   // arr[0] が T | undefined になる
    "exactOptionalPropertyTypes": true, // optional と undefined を区別
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

## 出力フォーマット

```markdown
## 型強化提案

### Step 1: any の排除
- 対象: `userController.ts:45`
- 現状: `(data: any) => ...`
- 変更案: `(data: unknown) => ...` + 型ガード追加
- 影響: 呼び出し側の修正は不要(unknownはanyからの代入を受ける)

### Step 2: Result型のDiscriminated Union化
- 対象: `api/result.ts`
- 影響箇所: 12ファイル(grep結果)
- 移行手順: ...

### tsconfig 改善(任意)
- `noUncheckedIndexedAccess: true` を提案。配列アクセスのnull安全性が向上
```

## 原則

- **段階的に。** 一気に厳格化すると修正コストが膨大になりレビュー困難。1 PR 1段階を推奨。
- **`as` は最後の手段。** 型ガードで表現できないか先に検討。
- **ライブラリ境界の型は要注意。** 外部APIレスポンスは `zod` 等で実行時検証も併用するとより堅牢。
- **過剰なジェネリクスは避ける。** 1箇所でしか使わない型を抽象化しない。
- **「型のためにコードを歪めない」。** ランタイム挙動の自然さを保つ。
