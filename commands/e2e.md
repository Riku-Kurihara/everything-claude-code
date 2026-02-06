---
description: Playwrightを使用してエンドツーエンドテストを生成・実行します。テストジャーニーを作成し、テストを実行し、スクリーンショット/ビデオ/トレースをキャプチャし、アーティファクトをアップロードします。
---

# E2Eコマンド

このコマンドは **e2e-runner** エージェントを呼び出して、Playwrightを使用してエンドツーエンドテストを生成、保守、実行します。

## このコマンドが行うこと

1. **テストジャーニーの生成** - ユーザーフロー用のPlaywrightテストを作成
2. **E2Eテストの実行** - ブラウザ全体でテストを実行
3. **アーティファクトのキャプチャ** - 失敗時にスクリーンショット、ビデオ、トレースをキャプチャ
4. **結果のアップロード** - HTMLレポートとJUnit XML
5. **不安定なテストの識別** - 不安定なテストを隔離

## いつ使用するか

以下の場合に `/e2e` を使用してください:
- 重要なユーザージャーニーのテスト (ログイン、取引、支払い)
- マルチステップフローがエンドツーエンドで機能することの確認
- UI相互作用とナビゲーションのテスト
- フロントエンドとバックエンド間の統合の検証
- 本番環境へのデプロイの準備

## 仕組み

e2e-runnerエージェントは以下を実行します:

1. **ユーザーフローを分析** してテストシナリオを特定
2. **Playwrightテストを生成** (ページオブジェクトモデルパターンを使用)
3. **複数のブラウザでテストを実行** (Chrome、Firefox、Safari)
4. **スクリーンショット、ビデオ、トレースで失敗をキャプチャ**
5. **結果とアーティファクトを含むレポートを生成**
6. **不安定なテストを特定** して修正を推奨

## 使用例

```
ユーザー: /e2e マーケット検索と表示フローをテスト

エージェント (e2e-runner):
# E2E テスト生成: マーケット検索と表示フロー

## 識別されたテストシナリオ

**ユーザージャーニー:** マーケット検索 → 結果表示 → マーケットをクリック → 詳細表示

**テストカバレッジ:**
1. マーケットページに移動
2. セマンティック検索を実行
3. 検索結果を確認
4. 最初の結果をクリック
5. マーケット詳細ページが読み込まれることを確認
6. チャートがレンダリングされることを確認

## 生成されたテストコード

```typescript
// tests/e2e/markets/search-and-view.spec.ts
import { test, expect } from '@playwright/test'
import { MarketsPage } from '../../pages/MarketsPage'
import { MarketDetailsPage } from '../../pages/MarketDetailsPage'

test.describe('マーケット検索と表示フロー', () => {
  test('ユーザーがマーケットを検索して詳細を表示できる', async ({ page }) => {
    // 1. マーケットページに移動
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // ページが読み込まれたことを確認
    await expect(page).toHaveTitle(/マーケット/)
    await expect(page.locator('h1')).toContainText('マーケット')

    // 2. セマンティック検索を実行
    await marketsPage.searchMarkets('選挙')

    // API レスポンスを待機
    await page.waitForResponse(resp =>
      resp.url().includes('/api/markets/search') && resp.status() === 200
    )

    // 3. 検索結果を確認
    const marketCards = marketsPage.marketCards
    await expect(marketCards.first()).toBeVisible()
    const resultCount = await marketCards.count()
    expect(resultCount).toBeGreaterThan(0)

    // 検索結果のスクリーンショットを撮影
    await page.screenshot({ path: 'artifacts/search-results.png' })

    // 4. 最初の結果をクリック
    const firstMarketTitle = await marketCards.first().textContent()
    await marketCards.first().click()

    // 5. マーケット詳細ページが読み込まれたことを確認
    await expect(page).toHaveURL(/\/markets\/[a-z0-9-]+/)

    const detailsPage = new MarketDetailsPage(page)
    await expect(detailsPage.marketName).toBeVisible()
    await expect(detailsPage.marketDescription).toBeVisible()

    // 6. チャートがレンダリングされることを確認
    await expect(detailsPage.priceChart).toBeVisible()

    // マーケット名が一致することを確認
    const detailsTitle = await detailsPage.marketName.textContent()
    expect(detailsTitle?.toLowerCase()).toContain(
      firstMarketTitle?.toLowerCase().substring(0, 20) || ''
    )

    // マーケット詳細のスクリーンショットを撮影
    await page.screenshot({ path: 'artifacts/market-details.png' })
  })

  test('検索結果なしの場合は空の状態を表示', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 存在しないマーケットを検索
    await marketsPage.searchMarkets('xyznonexistentmarket123456')

    // 空の状態を確認
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    await expect(page.locator('[data-testid="no-results"]')).toContainText(
      /結果がありません|マーケットがありません/i
    )

    const marketCount = await marketsPage.marketCards.count()
    expect(marketCount).toBe(0)
  })

  test('検索をクリアしてすべてのマーケットを再度表示できる', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // Initial market count
    const initialCount = await marketsPage.marketCards.count()

    // Perform search
    await marketsPage.searchMarkets('trump')
    await page.waitForLoadState('networkidle')

    // Verify filtered results
    const filteredCount = await marketsPage.marketCards.count()
    expect(filteredCount).toBeLessThan(initialCount)

    // Clear search
    await marketsPage.searchInput.clear()
    await page.waitForLoadState('networkidle')

    // Verify all markets shown again
    const finalCount = await marketsPage.marketCards.count()
    expect(finalCount).toBe(initialCount)
  })
})
```

## Running Tests

```bash
# Run the generated test
npx playwright test tests/e2e/markets/search-and-view.spec.ts

Running 3 tests using 3 workers

  ✓  [chromium] › search-and-view.spec.ts:5:3 › user can search markets and view details (4.2s)
  ✓  [chromium] › search-and-view.spec.ts:52:3 › search with no results shows empty state (1.8s)
  ✓  [chromium] › search-and-view.spec.ts:67:3 › can clear search and see all markets again (2.9s)

  3 passed (9.1s)

Artifacts generated:
- artifacts/search-results.png
- artifacts/market-details.png
- playwright-report/index.html
```

## Test Report

```
╔══════════════════════════════════════════════════════════════╗
║                    E2E Test Results                          ║
╠══════════════════════════════════════════════════════════════╣
║ Status:     ✅ ALL TESTS PASSED                              ║
║ Total:      3 tests                                          ║
║ Passed:     3 (100%)                                         ║
║ Failed:     0                                                ║
║ Flaky:      0                                                ║
║ Duration:   9.1s                                             ║
╚══════════════════════════════════════════════════════════════╝

Artifacts:
📸 Screenshots: 2 files
📹 Videos: 0 files (only on failure)
🔍 Traces: 0 files (only on failure)
📊 HTML Report: playwright-report/index.html

View report: npx playwright show-report
```

✅ E2E test suite ready for CI/CD integration!
```

## テストアーティファクト

テストが実行されるとき、以下のアーティファクトがキャプチャされます:

**すべてのテストで:**
- タイムラインと結果を含むHTMLレポート
- CI統合用のJUnit XML

**失敗時のみ:**
- 失敗した状態のスクリーンショット
- テストのビデオ記録
- トレースファイル (ステップバイステップ再生用のデバッグ)
- ネットワークログ
- コンソールログ

## アーティファクトの表示

```bash
# ブラウザでHTMLレポートを表示
npx playwright show-report

# 特定のトレースファイルを表示
npx playwright show-trace artifacts/trace-abc123.zip

# スクリーンショットはartifacts/ディレクトリに保存されます
open artifacts/search-results.png
```

## 不安定なテストの検出

テストが断続的に失敗する場合:

```
⚠️  不安定なテストが検出されました: tests/e2e/markets/trade.spec.ts

テストは10回中7回成功しました (成功率70%)

一般的な失敗:
"要素 '[data-testid="confirm-btn"]' を待機中にタイムアウト"

推奨修正:
1. 明示的な待機を追加: await page.waitForSelector('[data-testid="confirm-btn"]')
2. タイムアウトを増加: { timeout: 10000 }
3. コンポーネント内のレース条件をチェック
4. 要素がアニメーションで非表示になっていないか確認

隔離の推奨: 修正されるまでtest.fixme()としてマーク
```

## ブラウザ設定

テストはデフォルトで複数のブラウザで実行されます:
- ✅ Chromium (Desktop Chrome)
- ✅ Firefox (Desktop)
- ✅ WebKit (Desktop Safari)
- ✅ Mobile Chrome (オプション)

`playwright.config.ts` で設定してブラウザを調整してください。

## CI/CD統合

CI パイプラインに追加してください:

```yaml
# .github/workflows/e2e.yml
- name: Playwright をインストール
  run: npx playwright install --with-deps

- name: E2E テストを実行
  run: npx playwright test

- name: アーティファクトをアップロード
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

## PMX固有の重要フロー

PMXの場合、これらのE2Eテストを優先してください:

**🔴 重大 (常に成功する必要がある):**
1. ユーザーがウォレットを接続できる
2. ユーザーがマーケットを閲覧できる
3. ユーザーがマーケットを検索できる (セマンティック検索)
4. ユーザーがマーケット詳細を表示できる
5. ユーザーがトレードを実行できる (テスト資金で)
6. マーケットが正しく解決される
7. ユーザーが資金を引き出せる

**🟡 重要:**
1. マーケット作成フロー
2. ユーザープロフィール更新
3. リアルタイム価格更新
4. チャートレンダリング
5. フィルターとソート機能
6. モバイルレスポンシブレイアウト

## ベストプラクティス

**する:**
- ✅ 保守性のためページオブジェクトモデルを使用
- ✅ セレクターにdata-testid属性を使用
- ✅ 任意のタイムアウトではなくAPIレスポンスを待機
- ✅ 重要なユーザージャーニーをエンドツーエンドでテスト
- ✅ mainへのマージの前にテストを実行
- ✅ テストが失敗したときにアーティファクトをレビュー

**しない:**
- ❌ 脆弱なセレクター (CSSクラスは変更される可能性がある)
- ❌ 実装の詳細をテスト
- ❌ 本番環境に対してテストを実行
- ❌ 不安定なテストを無視
- ❌ 失敗時のアーティファクトレビューをスキップ
- ❌ すべてのエッジケースをE2Eでテスト (ユニットテストを使用)

## 重要な注記

**PMXにとって重大:**
- 実資金を含むE2Eテストはtestnet/stagingでのみ実行する必要があります
- 本番環境に対してトレーディングテストを実行しないでください
- 金融テストの場合は `test.skip(process.env.NODE_ENV === 'production')` を設定してください
- 小額のテスト資金を持つテストウォレットのみを使用してください

## 他のコマンドとの統合

- 詳細な分析には `/plan` を使用
- より高速で詳細なテストには `/tdd` を使用してユニットテストを実行
- 統合とユーザージャーニーテストには `/e2e` を使用
- テスト品質を検証するには `/code-review` を使用

## 関連エージェント

このコマンドは `e2e-runner` エージェントを呼び出します:
`~/.claude/agents/e2e-runner.md`

## クイックコマンド

```bash
# すべてのE2Eテストを実行
npx playwright test

# 特定のテストファイルを実行
npx playwright test tests/e2e/markets/search.spec.ts

# ヘッドモードで実行 (ブラウザを表示)
npx playwright test --headed

# テストをデバッグ
npx playwright test --debug

# テストコードを生成
npx playwright codegen http://localhost:3000

# レポートを表示
npx playwright show-report
```
