---
name: tdd-workflow
description: 新しい機能の作成、バグの修正、またはコードのリファクタリング時に、このスキルを使用してください。80% 以上のカバレッジを含むユニット、統合、E2E テストを備えたテスト駆動開発を強制します。
---

# テスト駆動開発ワークフロー

このスキルにより、すべてのコード開発が TDD 原則に従い、包括的なテストカバレッジを持つようになります。

## アクティベートのタイミング

- 新しい機能または機能の作成
- バグまたは問題の修正
- 既存コードのリファクタリング
- API エンドポイントの追加
- 新しいコンポーネントの作成

## コア原則

### 1. テストがコードの前に

常に最初にテストを書き、その後、テストを成功させるコードを実装します。

### 2. カバレッジ要件

- 最小 80% カバレッジ（ユニット + 統合 + E2E）
- すべてのエッジケースがカバーされている
- エラーシナリオがテストされている
- 境界条件が検証されている

### 3. テストタイプ

#### ユニットテスト

- 個別の関数とユーティリティ
- コンポーネントロジック
- 純粋関数
- ヘルパーとユーティリティ

#### 統合テスト

- API エンドポイント
- データベース操作
- サービス相互作用
- 外部 API 呼び出し

#### E2E テスト（Playwright）

- 重要なユーザーフロー
- 完全なワークフロー
- ブラウザオートメーション
- UI インタラクション

## TDD ワークフロー手順

### ステップ 1: ユーザージャーニーを記述する

```
[ロール]として、[アクション]したいので、[利点]を得たい

例：
ユーザーとして、市場をセマンティック検索したいので、正確なキーワードがなくても関連する市場を見つけることができます。
```

### ステップ 2: テストケースを生成

各ユーザージャーニーに対して、包括的なテストケースを作成します：

```typescript
describe('セマンティック検索', () => {
  it('クエリに関連する市場を返す', async () => {
    // テスト実装
  })

  it('空のクエリを適切に処理', async () => {
    // テストエッジケース
  })

  it('Redis が利用不可の場合、部分文字列検索にフォールバック', async () => {
    // フォールバック動作のテスト
  })

  it('結果を類似性スコアでソート', async () => {
    // ソートロジックをテスト
  })
})
```

### ステップ 3: テストを実行（失敗するはず）

```bash
npm test
# テストは失敗するはず - まだ実装していないため
```

### ステップ 4: コードを実装

テストで指示されたテストを通すための最小限のコードを書きます：

```typescript
// テストで指示される実装
export async function searchMarkets(query: string) {
  // ここに実装
}
```

### ステップ 5: テストを再度実行

```bash
npm test
# テストは現在成功するはず
```

### ステップ 6: リファクタリング

テストが成功している状態でコード品質を向上させます：
- 重複を削除
- 命名を改善
- パフォーマンスを最適化
- 読みやすさを向上

### ステップ 7: カバレッジを確認

```bash
npm run test:coverage
# 80% 以上のカバレッジが達成されたことを確認
```

## テストパターン

### ユニットテストパターン（Jest/Vitest）

```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('ボタンコンポーネント', () => {
  it('正しいテキストでレンダリング', () => {
    render(<Button>クリック</Button>)
    expect(screen.getByText('クリック')).toBeInTheDocument()
  })

  it('クリック時に onClick を呼び出す', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>クリック</Button>)

    fireEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('disabled prop が true の場合、無効になる', () => {
    render(<Button disabled>クリック</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### API 統合テストパターン

```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

describe('GET /api/markets', () => {
  it('市場を正常に返す', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
    expect(Array.isArray(data.data)).toBe(true)
  })

  it('クエリパラメータを検証', async () => {
    const request = new NextRequest('http://localhost/api/markets?limit=invalid')
    const response = await GET(request)

    expect(response.status).toBe(400)
  })

  it('データベースエラーを適切に処理', async () => {
    // データベース障害をモック
    const request = new NextRequest('http://localhost/api/markets')
    // エラー処理をテスト
  })
})
```

### E2E テストパターン（Playwright）

```typescript
import { test, expect } from '@playwright/test'

test('ユーザーが市場を検索およびフィルタリングできる', async ({ page }) => {
  // 市場ページに移動
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // ページが読み込まれたことを確認
  await expect(page.locator('h1')).toContainText('Markets')

  // 市場を検索
  await page.fill('input[placeholder="Search markets"]', 'election')

  // デバウンスと結果を待つ
  await page.waitForTimeout(600)

  // 検索結果が表示されたことを確認
  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // 結果に検索用語が含まれていることを確認
  const firstResult = results.first()
  await expect(firstResult).toContainText('election', { ignoreCase: true })

  // ステータスでフィルタリング
  await page.click('button:has-text("Active")')

  // フィルタリングされた結果を確認
  await expect(results).toHaveCount(3)
})

test('ユーザーが新しい市場を作成できる', async ({ page }) => {
  // 最初にログイン
  await page.goto('/creator-dashboard')

  // 市場作成フォームを入力
  await page.fill('input[name="name"]', 'Test Market')
  await page.fill('textarea[name="description"]', 'Test description')
  await page.fill('input[name="endDate"]', '2025-12-31')

  // フォームを送信
  await page.click('button[type="submit"]')

  // 成功メッセージを確認
  await expect(page.locator('text=Market created successfully')).toBeVisible()

  // 市場ページへのリダイレクトを確認
  await expect(page).toHaveURL(/\/markets\/test-market/)
})
```

## テストファイルの組織

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # ユニットテスト
│   │   └── Button.stories.tsx       # Storybook
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # 統合テスト
└── e2e/
    ├── markets.spec.ts               # E2E テスト
    ├── trading.spec.ts
    └── auth.spec.ts
```

## 外部サービスのモック

### Supabase モック

```typescript
jest.mock('@/lib/supabase', () => ({
  supabase: {
    from: jest.fn(() => ({
      select: jest.fn(() => ({
        eq: jest.fn(() => Promise.resolve({
          data: [{ id: 1, name: 'Test Market' }],
          error: null
        }))
      }))
    }))
  }
}))
```

### Redis モック

```typescript
jest.mock('@/lib/redis', () => ({
  searchMarketsByVector: jest.fn(() => Promise.resolve([
    { slug: 'test-market', similarity_score: 0.95 }
  ])),
  checkRedisHealth: jest.fn(() => Promise.resolve({ connected: true }))
}))
```

### OpenAI モック

```typescript
jest.mock('@/lib/openai', () => ({
  generateEmbedding: jest.fn(() => Promise.resolve(
    new Array(1536).fill(0.1) // モック 1536 次元埋め込み
  ))
}))
```

## テストカバレッジ検証

### カバレッジレポートを実行

```bash
npm run test:coverage
```

### カバレッジしきい値

```json
{
  "jest": {
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

## テストを避けるべき一般的な間違い

### 間違い: 実装の詳細をテスト

```typescript
// 内部状態をテストしない
expect(component.state.count).toBe(5)
```

### 正解: ユーザーに見える動作をテスト

```typescript
// ユーザーが見るものをテスト
expect(screen.getByText('Count: 5')).toBeInTheDocument()
```

### 間違い: ぜい弱なセレクタ

```typescript
// 簡単に壊れる
await page.click('.css-class-xyz')
```

### 正解: セマンティックセレクタ

```typescript
// 変更に強い
await page.click('button:has-text("Submit")')
await page.click('[data-testid="submit-button"]')
```

### 間違い: テスト分離なし

```typescript
// テストが互いに依存している
test('ユーザーを作成', () => { /* ... */ })
test('同じユーザーを更新', () => { /* 前のテストに依存 */ })
```

### 正解: 独立したテスト

```typescript
// 各テストが独自のデータをセットアップ
test('ユーザーを作成', () => {
  const user = createTestUser()
  // テストロジック
})

test('ユーザーを更新', () => {
  const user = createTestUser()
  // 更新ロジック
})
```

## 継続的テスト

### 開発中のウォッチモード

```bash
npm test -- --watch
# テストはファイル変更時に自動実行される
```

### プリコミットフック

```bash
# すべてのコミット前に実行
npm test && npm run lint
```

### CI/CD 統合

```yaml
# GitHub Actions
- name: テストを実行
  run: npm test -- --coverage
- name: カバレッジをアップロード
  uses: codecov/codecov-action@v3
```

## ベストプラクティス

1. **テストを最初に書く** - 常に TDD で
2. **テストごとに 1 つのアサーション** - 単一の動作に焦点
3. **説明的なテスト名** - テストされている内容を説明
4. **Arrange-Act-Assert** - 明確なテスト構造
5. **外部依存関係をモック** - ユニットテストを分離
6. **エッジケースをテスト** - null、undefined、empty、large
7. **エラーパスをテスト** - happy path だけでなく
8. **テストを高速に保つ** - ユニットテスト < 50ms
9. **テスト後にクリーンアップ** - 副作用なし
10. **カバレッジレポートを確認** - ギャップを特定

## 成功メトリクス

- 80% 以上のコードカバレッジを達成
- すべてのテストが成功（緑）
- スキップまたは無効化されたテストなし
- テスト実行が高速（ユニットテスト < 30 秒）
- E2E テストが重要なユーザーフローをカバー
- テストが本番環境の前にバグをキャッチ

---

**重要**: テストはオプションではありません。自信を持ってリファクタリングし、迅速に開発し、本番信頼性を実現するための安全ネットです。
