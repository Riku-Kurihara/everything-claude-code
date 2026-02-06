---
name: coding-standards
description: TypeScript、JavaScript、React、Node.js開発向けの普遍的なコーディング基準、ベストプラクティス、パターン。
---

# コーディング基準とベストプラクティス

すべてのプロジェクトに適用可能な普遍的なコーディング基準。

## コード品質の原則

### 1. 可読性を最優先に
- コードは書くより読まれる
- 明確な変数と関数名
- コメントより自己説明的なコードを優先
- 一貫した書式

### 2. KISS(Keep It Simple, Stupid)
- 機能する最もシンプルなソリューション
- 過剰エンジニアリングを回避
- 時期尚早な最適化をしない
- 理解しやすい > 巧妙なコード

### 3. DRY(Don't Repeat Yourself)
- 共通ロジックを関数に抽出
- 再利用可能なコンポーネント作成
- モジュール間でユーティリティを共有
- コピー&ペーストプログラミング回避

### 4. YAGNI(You Aren't Gonna Need It)
- 必要になる前に機能を構築しない
- 投機的な一般化を回避
- 必要な場合のみ複雑さを追加
- シンプルから始めて、必要に応じてリファクタ

## TypeScript/JavaScript基準

### 変数命名

```typescript
// ✅ GOOD: 記述的な名前
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ BAD: 不明確な名前
const q = 'election'
const flag = true
const x = 1000
```

### 関数命名

```typescript
// ✅ GOOD: 動詞-名詞パターン
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ BAD: 不明確または名詞のみ
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### イミュータビリティパターン(重要)

```typescript
// ✅ スプレッド演算子を常に使用
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 直接的なミューテーションは禁止
user.name = 'New Name'  // BAD
items.push(newItem)     // BAD
```

### エラーハンドリング

```typescript
// ✅ GOOD: 包括的なエラーハンドリング
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('Fetch failed:', error)
    throw new Error('Failed to fetch data')
  }
}

// ❌ BAD: エラーハンドリングなし
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### Async/Await ベストプラクティス

```typescript
// ✅ GOOD: 可能な場合は並列実行
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// ❌ BAD: 不要な連続実行
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 型安全性

```typescript
// ✅ GOOD: 適切な型
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

function getMarket(id: string): Promise<Market> {
  // 実装
}

// ❌ BAD: anyを使用
function getMarket(id: any): Promise<any> {
  // 実装
}
```

## Reactベストプラクティス

### コンポーネント構造

```typescript
// ✅ GOOD: 型付き関数コンポーネント
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// ❌ BAD: 型なし、構造不明確
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### カスタムフック

```typescript
// ✅ GOOD: 再利用可能なカスタムフック
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}

// 使用方法
const debouncedQuery = useDebounce(searchQuery, 500)
```

### 状態管理

```typescript
// ✅ GOOD: 適切な状態更新
const [count, setCount] = useState(0)

// 前の状態に基づく状態更新に関数形式を使用
setCount(prev => prev + 1)

// ❌ BAD: 直接状態参照
setCount(count + 1)  // 非同期シナリオで古い可能性
```

### 条件付きレンダリング

```typescript
// ✅ GOOD: 明確な条件付きレンダリング
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ BAD: ternary地獄
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API設計基準

### REST API慣例

```
GET    /api/markets              # すべてのマーケット取得
GET    /api/markets/:id          # 特定のマーケット取得
POST   /api/markets              # 新しいマーケット作成
PUT    /api/markets/:id          # マーケット完全更新
PATCH  /api/markets/:id          # マーケット部分更新
DELETE /api/markets/:id          # マーケット削除

# フィルタリング用クエリパラメータ
GET /api/markets?status=active&limit=10&offset=0
```

### レスポンス形式

```typescript
// ✅ GOOD: 一貫したレスポンス構造
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}

// 成功レスポンス
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 }
})

// エラーレスポンス
return NextResponse.json({
  success: false,
  error: 'Invalid request'
}, { status: 400 })
```

### 入力検証

```typescript
import { z } from 'zod'

// ✅ GOOD: スキーマ検証
const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime(),
  categories: z.array(z.string()).min(1)
})

export async function POST(request: Request) {
  const body = await request.json()

  try {
    const validated = CreateMarketSchema.parse(body)
    // 検証済みデータで進行
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        success: false,
        error: 'Validation failed',
        details: error.errors
      }, { status: 400 })
    }
  }
}
```

## ファイル構成

### プロジェクト構造

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # APIルート
│   ├── markets/           # マーケットページ
│   └── (auth)/           # 認証ページ(ルートグループ)
├── components/            # Reactコンポーネント
│   ├── ui/               # 汎用UIコンポーネント
│   ├── forms/            # フォームコンポーネント
│   └── layouts/          # レイアウトコンポーネント
├── hooks/                # カスタムReactフック
├── lib/                  # ユーティリティと設定
│   ├── api/             # APIクライアント
│   ├── utils/           # ヘルパー関数
│   └── constants/       # 定数
├── types/                # TypeScript型
└── styles/              # グローバルスタイル
```

### ファイル命名

```
components/Button.tsx          # PascalCase コンポーネント
hooks/useAuth.ts              # camelCase 'use'プレフィックス
lib/formatDate.ts             # camelCase ユーティリティ
types/market.types.ts         # camelCase .types接尾辞
```

## コメントとドキュメント

### コメント時期

```typescript
// ✅ GOOD: WHYを説明、WHATではなく
// 停止中のAPI負荷を避けるため指数バックオフ使用
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 大規模配列でのパフォーマンスのためmutationを意図的に使用
items.push(newItem)

// ❌ BAD: 自明なことを述べる
// カウンターを1増加
count++

// ユーザーの名前を名前に設定
name = user.name
```

### 公開API用JSDoc

```typescript
/**
 * 意味的類似度を使用してマーケットを検索します。
 *
 * @param query - 自然言語検索クエリ
 * @param limit - 結果の最大数(デフォルト: 10)
 * @returns 類似度スコアでソートされたマーケット配列
 * @throws {Error} OpenAI APIが失敗またはRedis不可用の場合
 *
 * @example
 * ```typescript
 * const results = await searchMarkets('election', 5)
 * console.log(results[0].name) // "Trump vs Biden"
 * ```
 */
export async function searchMarkets(
  query: string,
  limit: number = 10
): Promise<Market[]> {
  // 実装
}
```

## パフォーマンスベストプラクティス

### メモ化

```typescript
import { useMemo, useCallback } from 'react'

// ✅ GOOD: 高コスト計算をメモ化
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ GOOD: コールバックをメモ化
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### 遅延ロード

```typescript
import { lazy, Suspense } from 'react'

// ✅ GOOD: 重いコンポーネントを遅延ロード
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### データベースクエリ

```typescript
// ✅ GOOD: 必要なカラムのみ選択
const { data } = await supabase
  .from('markets')
  .select('id, name, status')
  .limit(10)

// ❌ BAD: すべて選択
const { data } = await supabase
  .from('markets')
  .select('*')
```

## テスト基準

### テスト構造(AAAパターン)

```typescript
test('calculates similarity correctly', () => {
  // Arrange
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert
  expect(similarity).toBe(0)
})
```

### テスト命名

```typescript
// ✅ GOOD: 記述的なテスト名
test('returns empty array when no markets match query', () => { })
test('throws error when OpenAI API key is missing', () => { })
test('falls back to substring search when Redis unavailable', () => { })

// ❌ BAD: 曖昧なテスト名
test('works', () => { })
test('test search', () => { })
```

## コード臭い検出

これらのアンチパターンに注意:

### 1. 長い関数
```typescript
// ❌ BAD: 50行以上の関数
function processMarketData() {
  // 100行のコード
}

// ✅ GOOD: より小さな関数に分割
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 深いネスト
```typescript
// ❌ BAD: 5レベル以上のネスト
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 何かをする
        }
      }
    }
  }
}

// ✅ GOOD: 早期リターン
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 何かをする
```

### 3. マジックナンバー
```typescript
// ❌ BAD: 説明なしの数字
if (retryCount > 3) { }
setTimeout(callback, 500)

// ✅ GOOD: 名前付き定数
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500

if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

**覚えておこう**: コード品質は非交渉です。明確で保守可能なコードは、迅速な開発と自信を持ったリファクタリングを実現します。
