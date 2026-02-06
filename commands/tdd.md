---
description: テスト駆動開発ワークフローを強制します。インターフェースをスキャフォルド、テストを最初に生成、その後最小限のコードを実装して成功させます。80%+ カバレッジを確認してください。
---

# TDD コマンド

このコマンドは **tdd-guide** エージェントを呼び出してテスト駆動開発方法論を強制します。

## このコマンドが行うこと

1. **スキャフォルドインターフェース** - 型/インターフェースを最初に定義
2. **テストを最初に生成** - 失敗するテストを書く (赤)
3. **最小限のコードを実装** - 成功するためのちょうど十分なコードを書く (緑)
4. **リファクタ** - テストが緑のままでコードを改善 (リファクタ)
5. **カバレッジを検証** - 80%+ のテストカバレッジを確認

## いつ使用するか

以下の場合に `/tdd` を使用してください:
- 新しい機能を実装するとき
- 新しい関数/コンポーネントを追加
- バグ修正 (最初にバグを再現するテストを書く)
- 既存コードをリファクタリング
- 重要なビジネスロジックを構築

## 仕組み

tdd-guide エージェントは以下を実行します:

1. **入出力用のインターフェースを定義**
2. **失敗するテストを書く** (コードがまだ存在しないため)
3. **テストを実行** して正しい理由で失敗することを確認
4. **最小限の実装を書く** テストに成功させるために
5. **テストを実行** して成功することを確認
6. **リファクタ** テストが緑のままでコード
7. **カバレッジをチェック** し 80% 以下の場合はテストを追加

## TDD サイクル

```
赤 → 緑 → リファクタ → 繰り返す

赤:        失敗するテストを書く
緑:        成功するための最小限のコードを書く
リファクタ: コードを改善、テストを成功させたままにする
繰り返す:   次の機能/シナリオ
```

## Example Usage

```
User: /tdd I need a function to calculate market liquidity score

Agent (tdd-guide):
# TDD Session: Market Liquidity Score Calculator

## Step 1: Define Interface (SCAFFOLD)

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  // TODO: Implementation
  throw new Error('Not implemented')
}
```

## Step 2: Write Failing Test (RED)

```typescript
// lib/liquidity.test.ts
import { calculateLiquidityScore } from './liquidity'

describe('calculateLiquidityScore', () => {
  it('should return high score for liquid market', () => {
    const market = {
      totalVolume: 100000,
      bidAskSpread: 0.01,
      activeTraders: 500,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeGreaterThan(80)
    expect(score).toBeLessThanOrEqual(100)
  })

  it('should return low score for illiquid market', () => {
    const market = {
      totalVolume: 100,
      bidAskSpread: 0.5,
      activeTraders: 2,
      lastTradeTime: new Date(Date.now() - 86400000) // 1 day ago
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeLessThan(30)
    expect(score).toBeGreaterThanOrEqual(0)
  })

  it('should handle edge case: zero volume', () => {
    const market = {
      totalVolume: 0,
      bidAskSpread: 0,
      activeTraders: 0,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBe(0)
  })
})
```

## Step 3: Run Tests - Verify FAIL

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ should return high score for liquid market (2 ms)
    Error: Not implemented

1 test failed, 0 passed
```

✅ Tests fail as expected. Ready to implement.

## Step 4: Implement Minimal Code (GREEN)

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  // Handle zero volume edge case
  if (market.totalVolume === 0) {
    return 0
  }

  // Calculate component scores (0-100 scale)
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  // Recent activity bonus
  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  // Weighted average
  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100) // Clamp to 0-100
}
```

## Step 5: Run Tests - Verify PASS

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

✅ All tests passing!

## Step 6: Refactor (IMPROVE)

```typescript
// lib/liquidity.ts - Refactored with constants and better readability
const WEIGHTS = {
  VOLUME: 0.4,
  SPREAD: 0.3,
  TRADERS: 0.2,
  RECENCY: 0.1,
} as const

const SCALE_FACTORS = {
  VOLUME: 1000,
  SPREAD: 1000,
  TRADERS: 10,
  RECENCY_PENALTY: 10,
} as const

function clamp(value: number, min: number, max: number): number {
  return Math.min(Math.max(value, min), max)
}

export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) return 0

  const volumeScore = Math.min(market.totalVolume / SCALE_FACTORS.VOLUME, 100)
  const spreadScore = clamp(100 - (market.bidAskSpread * SCALE_FACTORS.SPREAD), 0, 100)
  const traderScore = Math.min(market.activeTraders / SCALE_FACTORS.TRADERS, 100)

  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = clamp(100 - (hoursSinceLastTrade * SCALE_FACTORS.RECENCY_PENALTY), 0, 100)

  const weightedScore =
    volumeScore * WEIGHTS.VOLUME +
    spreadScore * WEIGHTS.SPREAD +
    traderScore * WEIGHTS.TRADERS +
    recencyScore * WEIGHTS.RECENCY

  return clamp(weightedScore, 0, 100)
}
```

## Step 7: Verify Tests Still Pass

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

✅ Refactoring complete, tests still passing!

## Step 8: Check Coverage

```bash
npm test -- --coverage lib/liquidity.test.ts

File           | % Stmts | % Branch | % Funcs | % Lines
---------------|---------|----------|---------|--------
liquidity.ts   |   100   |   100    |   100   |   100

Coverage: 100% ✅ (Target: 80%)
```

✅ TDD session complete!
```

## TDD ベストプラクティス

**すべき:**
- ✅ 実装前に、まずテストを書く
- ✅ テストを実行して、実装前に失敗することを確認
- ✅ テストを成功させるための最小限のコードを書く
- ✅ テストが緑になった後にのみリファクタリング
- ✅ エッジケースとエラーシナリオを追加
- ✅ 80%+ カバレッジを目指す (重要なコードは 100%)

**すべきでない:**
- ❌ テストの前に実装を書く
- ❌ 各変更後のテスト実行をスキップ
- ❌ 一度に多くのコードを書く
- ❌ 失敗するテストを無視
- ❌ 実装の詳細をテスト (動作をテスト)
- ❌ すべてをモック化 (統合テストを優先)

## 含めるテストタイプ

**ユニットテスト** (関数レベル):
- ハッピーパスシナリオ
- エッジケース (空、null、最大値)
- エラーコンディション
- 境界値

**統合テスト** (コンポーネントレベル):
- API エンドポイント
- データベース操作
- 外部サービス呼び出し
- React コンポーネント with フック

**E2E テスト** (`/e2e` コマンドを使用):
- 重要なユーザーフロー
- マルチステッププロセス
- フルスタック統合

## カバレッジ要件

- すべてのコードに対して **80% 以上**
- 以下に対して **100% が必須**:
  - 財務計算
  - 認証ロジック
  - セキュリティクリティカルなコード
  - コアビジネスロジック

## 重要な注記

**必須**: テストは実装の前に書かなければなりません。TDD サイクルは:

1. **RED** - 失敗するテストを書く
2. **GREEN** - 成功するために実装
3. **REFACTOR** - コードを改善

RED フェーズをスキップしないでください。テストの前にコードを書かないでください。

## 他のコマンドとの統合

- 最初に `/plan` を使用して何を構築するかを理解
- `/tdd` を使用してテストで実装
- ビルドエラーが発生した場合は `/build-and-fix` を使用
- 実装をレビューするには `/code-review` を使用
- カバレッジを検証するには `/test-coverage` を使用

## 関連エージェント

このコマンドは以下の場所の `tdd-guide` エージェントを呼び出します:
`~/.claude/agents/tdd-guide.md`

また、以下の場所の `tdd-workflow` スキルを参照できます:
`~/.claude/skills/tdd-workflow/`
