---
name: security-review
description: 認証、ユーザー入力処理、シークレット処理、APIエンドポイント作成、支払い/機密機能実装時に使用。包括的なセキュリティチェックリストとパターンを提供。
---

# セキュリティレビュースキル

このスキルはすべてのコードがセキュリティベストプラクティスに従い、潜在的な脆弱性を識別することを保証します。

## アクティベーション時期

- 認証または認可を実装するとき
- ユーザー入力またはファイルアップロードを処理するとき
- 新しいAPIエンドポイントを作成するとき
- シークレットまたは認証情報を処理するとき
- 支払い機能を実装するとき
- 機密データを保存または送信するとき
- サードパーティAPIを統合するとき

## セキュリティチェックリスト

### 1. シークレット管理

#### してはいけないこと

```typescript
const apiKey = "sk-proj-xxxxx"  // ハードコードされたシークレット
const dbPassword = "password123" // ソースコード内
```

#### すべきこと

```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// シークレットが存在することを確認
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

#### 確認ステップ

- [ ] ハードコードされたAPIキー、トークン、パスワードなし
- [ ] すべてのシークレットが環境変数に存在
- [ ] `.env.local`が.gitignoreに含まれている
- [ ] gitの履歴にシークレットなし
- [ ] 本番環境のシークレットがホスティングプラットフォーム（Vercel、Railway）にある

### 2. 入力検証

#### ユーザー入力を常に検証

```typescript
import { z } from 'zod'

// 検証スキーマを定義
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 処理前に検証
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

#### ファイルアップロード検証

```typescript
function validateFileUpload(file: File) {
  // サイズチェック (5MBまで)
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('File too large (max 5MB)')
  }

  // タイプチェック
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('Invalid file type')
  }

  // 拡張子チェック
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('Invalid file extension')
  }

  return true
}
```

#### 確認ステップ

- [ ] すべてのユーザー入力がスキーマで検証されている
- [ ] ファイルアップロードが制限されている（サイズ、タイプ、拡張子）
- [ ] ユーザー入力がクエリで直接使用されていない
- [ ] ホワイトリスト検証（ブラックリストではなく）
- [ ] エラーメッセージが機密情報をリークしていない

### 3. SQLインジェクション防止

#### してはいけないこと

```typescript
// 危険 - SQLインジェクション脆弱性
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### すべきこと

```typescript
// 安全 - パラメータ化クエリ
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// または生SQLで
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

#### 確認ステップ

- [ ] すべてのデータベースクエリがパラメータ化されている
- [ ] SQLで文字列連結がない
- [ ] ORM/クエリビルダーが正しく使用されている
- [ ] Supabaseクエリが適切にサニタイズされている

### 4. 認証と認可

#### JWTトークン処理

```typescript
// 誤り：localStorage（XSS脆弱性）
localStorage.setItem('token', token)

// 正解：httpOnlyクッキー
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 認可チェック

```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 常に認可を最初に検証
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 403 }
    )
  }

  // 削除を続行
  await db.users.delete({ where: { id: userId } })
}
```

#### 行レベルセキュリティ (Supabase)

```sql
-- すべてのテーブルでRLSを有効化
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- ユーザーは自分のデータのみを表示可能
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- ユーザーは自分のデータのみを更新可能
CREATE POLICY "Users update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

#### 確認ステップ

- [ ] トークンはhttpOnlyクッキーに保存（localStorageではなく）
- [ ] 機密操作の前に認可チェック
- [ ] Supabaseで行レベルセキュリティが有効
- [ ] ロールベースアクセス制御が実装されている
- [ ] セッション管理がセキュア

### 5. XSS防止

#### HTMLをサニタイズ

```typescript
import DOMPurify from 'isomorphic-dompurify'

// ユーザー提供のHTMLを常にサニタイズ
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### コンテンツセキュリティポリシー

```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s{2,}/g, ' ').trim()
  }
]
```

#### 確認ステップ

- [ ] ユーザー提供のHTMLがサニタイズされている
- [ ] CSPヘッダーが設定されている
- [ ] 検証されていない動的コンテンツのレンダリングがない
- [ ] Reactの組み込みXSS保護が使用されている

### 6. CSRF保護

#### CSRFトークン

```typescript
import { csrf } from '@/lib/csrf'

export async function POST(request: Request) {
  const token = request.headers.get('X-CSRF-Token')

  if (!csrf.verify(token)) {
    return NextResponse.json(
      { error: 'Invalid CSRF token' },
      { status: 403 }
    )
  }

  // リクエストを処理
}
```

#### SameSiteクッキー

```typescript
res.setHeader('Set-Cookie',
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`)
```

#### 確認ステップ

- [ ] 状態変更操作にCSRFトークンが含まれている
- [ ] すべてのクッキーにSameSite=Strictが設定されている
- [ ] ダブルサブミットクッキーパターンが実装されている

### 7. レート制限

#### APIレート制限

```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15分
  max: 100, // ウィンドウごとに100リクエスト
  message: 'Too many requests'
})

// ルートに適用
app.use('/api/', limiter)
```

#### 高コスト操作

```typescript
// 検索に対する強力なレート制限
const searchLimiter = rateLimit({
  windowMs: 60 * 1000, // 1分
  max: 10, // 1分あたり10リクエスト
  message: 'Too many search requests'
})

app.use('/api/search', searchLimiter)
```

#### 確認ステップ

- [ ] すべてのAPIエンドポイントにレート制限がある
- [ ] 高コスト操作により厳しい制限がある
- [ ] IPベースのレート制限
- [ ] ユーザーベースのレート制限（認証済み）

### 8. 機密データ露出

#### ログ出力

```typescript
// 誤り：機密データをログ出力
console.log('User login:', { email, password })
console.log('Payment:', { cardNumber, cvv })

// 正解：機密データをマスク
console.log('User login:', { email, userId })
console.log('Payment:', { last4: card.last4, userId })
```

#### エラーメッセージ

```typescript
// 誤り：内部詳細を露出
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// 正解：ジェネリックエラーメッセージ
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

#### 確認ステップ

- [ ] パスワード、トークン、シークレットがログに含まれていない
- [ ] エラーメッセージがユーザーに対してジェネリック
- [ ] 詳細なエラーはサーバーログのみ
- [ ] スタックトレースがユーザーに露出していない

### 9. ブロックチェーンセキュリティ (Solana)

#### ウォレット検証

```typescript
import { verify } from '@solana/web3.js'

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string
) {
  try {
    const isValid = verify(
      Buffer.from(message),
      Buffer.from(signature, 'base64'),
      Buffer.from(publicKey, 'base64')
    )
    return isValid
  } catch (error) {
    return false
  }
}
```

#### トランザクション検証

```typescript
async function verifyTransaction(transaction: Transaction) {
  // 受信者を検証
  if (transaction.to !== expectedRecipient) {
    throw new Error('Invalid recipient')
  }

  // 金額を検証
  if (transaction.amount > maxAmount) {
    throw new Error('Amount exceeds limit')
  }

  // ユーザーが十分な残高を持っていることを検証
  const balance = await getBalance(transaction.from)
  if (balance < transaction.amount) {
    throw new Error('Insufficient balance')
  }

  return true
}
```

#### 確認ステップ

- [ ] ウォレット署名が検証されている
- [ ] トランザクション詳細が検証されている
- [ ] トランザクション前に残高チェックが行われている
- [ ] ブラインドトランザクション署名がない

### 10. 依存関係セキュリティ

#### 定期的な更新

```bash
# 脆弱性をチェック
npm audit

# 自動修正可能な問題を修正
npm audit fix

# 依存関係を更新
npm update

# アウトデートパッケージをチェック
npm outdated
```

#### ロックファイル

```bash
# ロックファイルを常にコミット
git add package-lock.json

# 再現可能なビルドのためにCI/CDで使用
npm ci  # npm installの代わり
```

#### 確認ステップ

- [ ] 依存関係が最新
- [ ] 既知の脆弱性なし（npm auditクリーン）
- [ ] ロックファイルがコミットされている
- [ ] GitHubでDependabotが有効
- [ ] 定期的なセキュリティ更新

## セキュリティテスト

### 自動化されたセキュリティテスト

```typescript
// 認証をテスト
test('requires authentication', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

// 認可をテスト
test('requires admin role', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})

// 入力検証をテスト
test('rejects invalid input', async () => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ email: 'not-an-email' })
  })
  expect(response.status).toBe(400)
})

// レート制限をテスト
test('enforces rate limits', async () => {
  const requests = Array(101).fill(null).map(() =>
    fetch('/api/endpoint')
  )

  const responses = await Promise.all(requests)
  const tooManyRequests = responses.filter(r => r.status === 429)

  expect(tooManyRequests.length).toBeGreaterThan(0)
})
```

## 本番環境デプロイ前セキュリティチェックリスト

本番環境にデプロイする前に：

- [ ] **シークレット**: ハードコードされたシークレットなし、すべて環境変数に
- [ ] **入力検証**: すべてのユーザー入力が検証されている
- [ ] **SQLインジェクション**: すべてのクエリがパラメータ化されている
- [ ] **XSS**: ユーザーコンテンツがサニタイズされている
- [ ] **CSRF**: 保護が有効
- [ ] **認証**: 適切なトークン処理
- [ ] **認可**: ロールチェックが機能
- [ ] **レート制限**: すべてのエンドポイントで有効
- [ ] **HTTPS**: 本番環境で強制
- [ ] **セキュリティヘッダー**: CSP、X-Frame-Options設定済み
- [ ] **エラーハンドリング**: エラーに機密データなし
- [ ] **ログ出力**: ログに機密データなし
- [ ] **依存関係**: 最新、脆弱性なし
- [ ] **行レベルセキュリティ**: Supabaseで有効
- [ ] **CORS**: 適切に設定
- [ ] **ファイルアップロード**: 検証済み（サイズ、タイプ）
- [ ] **ウォレット署名**: 検証済み（ブロックチェーン）

## リソース

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**覚えておいてください**：セキュリティはオプションではありません。1つの脆弱性がプラットフォーム全体を危険にさらす可能性があります。確信が持てない場合は、注意を優先してください。
