---
name: doc-gen
description: 関数・クラスのインラインドキュメント(JSDoc/TSDoc/docstring/Javadoc/YARD/godoc/rustdoc)を生成する。docstring・JSDoc・コメント追加の依頼で使う。
---

# Documentation Generation

関数・クラスにインラインドキュメントを生成する。

## サポートする形式

| 言語 | デフォルト形式 |
| :--- | :--- |
| JavaScript | JSDoc |
| TypeScript | TSDoc (`@param` の型はTypeScript型に任せる) |
| Python | Google スタイル docstring(またはNumPy/Sphinx、既存に合わせる) |
| Java | Javadoc |
| Ruby | YARD |
| Go | godoc コメント |
| Rust | rustdoc (`///`) |

既存ファイルにドキュメントがあればそのスタイルを踏襲する。新規の場合はプロジェクト言語の標準を使う。

## 含めるべき要素

最低限:
- **概要(1行)** — 何をする関数/クラスか
- **パラメータ** — 各引数の意味、許容範囲、null可否
- **戻り値** — 何を返すか、エラー時の値

推奨:
- **例外/エラー** — どんなときに throw / Err を返すか
- **副作用** — DB書き込み、ファイル出力、外部API呼び出し等
- **使用例** — 短く実用的なコード片
- **計算量** — O(n) 等(非自明な場合)
- **関連** — 関連関数へのリンク

## 出力例

### TypeScript (TSDoc)

```typescript
/**
 * 商品価格に会員ランクに応じた割引を適用する。
 *
 * @param price - 元の価格(0以上の整数、円)
 * @param member - 会員情報。`null` の場合は割引なし
 * @returns 割引適用後の価格(円、小数点以下切り捨て)
 * @throws {Error} `price` が負の値の場合
 *
 * @example
 * ```ts
 * calculateDiscount(1000, { tier: 'vip' }); // 800
 * calculateDiscount(1000, null);             // 1000
 * ```
 */
function calculateDiscount(price: number, member: Member | null): number { ... }
```

### Python (Google スタイル)

```python
def calculate_discount(price: int, member: Member | None) -> int:
    """商品価格に会員ランクに応じた割引を適用する。

    Args:
        price: 元の価格(0以上の整数、円)。
        member: 会員情報。None の場合は割引なし。

    Returns:
        割引適用後の価格(円、小数点以下切り捨て)。

    Raises:
        ValueError: price が負の値の場合。

    Example:
        >>> calculate_discount(1000, Member(tier='vip'))
        800
    """
```

## 原則

- **コードを読めば分かることは書かない。** `// iをインクリメント` のような冗長コメントは生成しない。意図・契約・前提を書く。
- **嘘を書かない。** 実装を読んで確認した内容だけを書く。推測で `@throws` を増やさない。
- **型情報はTypeScript側に任せる。** TSDocで `{string}` のような型注釈を重複させない。
- **既存スタイルに合わせる。** 1ファイル内で形式が混ざるのは避ける。
- **日本語/英語はプロジェクト方針に従う。** 不明なら確認する。
