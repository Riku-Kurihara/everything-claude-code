# TypeScript/JavaScript コーディングスタイル

> このファイルは [common/coding-style.md](../common/coding-style.md) を TypeScript/JavaScript 特有の内容で拡張します。

## イミュータビリティ

イミュータブルな更新にスプレッド演算子を使用：

```typescript
// 間違い: ミューテーション
function updateUser(user, name) {
  user.name = name  // ミューテーション！
  return user
}

// 正しい: イミュータビリティ
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

## エラーハンドリング

async/await と try-catch を使用：

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('Operation failed:', error)
  throw new Error('Detailed user-friendly message')
}
```

## 入力検証

スキーマベースの検証に Zod を使用：

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## Console.log

- 本番コードに `console.log` ステートメントなし
- 代わりに適切なロギングライブラリを使用
- フックの自動検出については参照
