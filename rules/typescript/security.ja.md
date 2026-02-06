# TypeScript/JavaScript セキュリティ

> このファイルは [common/security.md](../common/security.md) を TypeScript/JavaScript 特有の内容で拡張します。

## シークレット管理

```typescript
// しない: ハードコードされたシークレット
const apiKey = "sk-proj-xxxxx"

// する: 環境変数
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

## エージェントサポート

- 包括的なセキュリティ監査には **security-reviewer** スキルを使用してください
