# 検証ループスキル

Claude Code セッションの包括的な検証システム。

## 使用時期

このスキルを呼び出す：
- 機能の完了または重要なコード変更後
- PR を作成する前
- 品質ゲートが通ることを確認したい場合
- リファクタリング後

## 検証フェーズ

### フェーズ 1: ビルド検証

```bash
# プロジェクトがビルドされているかを確認
npm run build 2>&1 | tail -20
# または
pnpm build 2>&1 | tail -20
```

ビルドが失敗した場合は停止して修正してください。

### フェーズ 2: 型チェック

```bash
# TypeScript プロジェクト
npx tsc --noEmit 2>&1 | head -30

# Python プロジェクト
pyright . 2>&1 | head -30
```

すべての型エラーをレポート。続行する前に重大なものを修正してください。

### フェーズ 3: Lint チェック

```bash
# JavaScript/TypeScript
npm run lint 2>&1 | head -30

# Python
ruff check . 2>&1 | head -30
```

### フェーズ 4: テストスイート

```bash
# カバレッジを使用してテストを実行
npm run test -- --coverage 2>&1 | tail -50

# カバレッジしきい値を確認
# 目標: 最小 80%
```

レポート：
- テスト総数：X
- 成功：X
- 失敗：X
- カバレッジ：X%

### フェーズ 5: セキュリティスキャン

```bash
# シークレットを確認
grep -rn "sk-" --include="*.ts" --include="*.js" . 2>/dev/null | head -10
grep -rn "api_key" --include="*.ts" --include="*.js" . 2>/dev/null | head -10

# console.log を確認
grep -rn "console.log" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | head -10
```

### フェーズ 6: 差分レビュー

```bash
# 何が変わったかを表示
git diff --stat
git diff HEAD~1 --name-only
```

各変更ファイルをレビュー：
- 意図しない変更
- エラーハンドリングの欠落
- 潜在的なエッジケース

## 出力形式

すべてのフェーズを実行した後、検証レポートを生成します：

```
検証レポート
==============

ビルド:     [PASS/FAIL]
型:        [PASS/FAIL] (X エラー)
Lint:      [PASS/FAIL] (X 警告)
テスト:    [PASS/FAIL] (X/Y 成功、Z% カバレッジ)
セキュリティ: [PASS/FAIL] (X 問題)
差分:      [X ファイル変更]

全体:      [PR 準備完了/準備未完了]

修正する問題:
1. ...
2. ...
```

## 継続モード

長いセッションの場合は、15 分ごと、または大きな変更後に検証を実行します：

```markdown
メンタルチェックポイントを設定：
- 各関数の完了後
- コンポーネントの完了後
- 次のタスクに移動する前

実行: /verify
```

## フックとの統合

このスキルは PostToolUse フックを補完していますが、より深い検証を提供します。
フックはすぐに問題をキャッチしますが、このスキルは包括的なレビューを提供します。
