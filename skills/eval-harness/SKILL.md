---
name: eval-harness
description: Claude Code セッション向けの正式な評価フレームワークで、eval駆動開発（EDD）原則を実装しています。
tools: Read, Write, Edit, Bash, Grep, Glob
---

# Eval Harness スキル

Claude Code セッション向けの正式な評価フレームワークで、eval駆動開発（EDD）原則を実装しています。

## 哲学

Eval駆動開発は、evalをAI開発のユニットテストとして扱います:
- 実装の前に期待される動作を定義
- 開発中に継続的にevalを実行
- 各変更で回帰を追跡
- pass@kメトリクスで信頼性を測定

## Eval タイプ

### 能力Eval
Claudeが以前にできなかったことができるかをテスト:
```markdown
[CAPABILITY EVAL: feature-name]
タスク: Claudeが達成すべきことの説明
成功基準:
  - [ ] 基準 1
  - [ ] 基準 2
  - [ ] 基準 3
期待される出力: 予想される結果の説明
```

### 回帰Eval
変更が既存機能を壊さないことを確認:
```markdown
[REGRESSION EVAL: feature-name]
ベースライン: SHAまたはチェックポイント名
テスト:
  - existing-test-1: PASS/FAIL
  - existing-test-2: PASS/FAIL
  - existing-test-3: PASS/FAIL
結果: X/Y成功（以前は Y/Y）
```

## グレーダー タイプ

### 1. コードベースのグレーダー
コードを使用した確定的なチェック:
```bash
# ファイルに期待されるパターンが含まれているかをチェック
grep -q "export function handleAuth" src/auth.ts && echo "PASS" || echo "FAIL"

# テストが成功するかをチェック
npm test -- --testPathPattern="auth" && echo "PASS" || echo "FAIL"

# ビルドが成功するかをチェック
npm run build && echo "PASS" || echo "FAIL"
```

### 2. モデルベースのグレーダー
Claudeを使用してオープンエンドの出力を評価:
```markdown
[MODEL GRADER PROMPT]
以下のコード変更を評価:
1. 述べられた問題を解決しているか?
2. 適切に構造化されているか?
3. エッジケースは処理されているか?
4. エラーハンドリングは適切か?

スコア: 1-5（1=悪い、5=優秀）
理由: [説明]
```

### 3. 人間グレーダー
手動レビューのためにフラグ:
```markdown
[HUMAN REVIEW REQUIRED]
変更: 何が変わったかの説明
理由: 人間レビューが必要な理由
リスク レベル: 低/中/高
```

## メトリクス

### pass@k
「k回の試行で少なくとも1回の成功」
- pass@1: 最初の試行の成功率
- pass@3: 3回の試行内での成功
- 典型的なターゲット: pass@3 > 90%

### pass^k
「k回すべての試行が成功」
- より信頼性の高い基準
- pass^3: 3回連続の成功
- 重大経路に使用

## Eval ワークフロー

### 1. 定義（コード前）
```markdown
## EVAL DEFINITION: feature-xyz

### 能力Eval
1. 新しいユーザーアカウントを作成できる
2. メールアドレス形式を検証できる
3. パスワードを安全にハッシュ化できる

### 回帰Eval
1. 既存のログインが機能する
2. セッション管理は変わらない
3. ログアウトフロー完全

### 成功メトリクス
- 能力evalで pass@3 > 90%
- 回帰evalで pass^3 = 100%
```

### 2. 実装
定義されたevalを成功させるコードを書きます。

### 3. 評価
```bash
# 能力evalを実行
[各能力evalを実行、PASS/FAILを記録]

# 回帰evalを実行
npm test -- --testPathPattern="existing"

# レポートを生成
```

### 4. レポート
```markdown
EVAL REPORT: feature-xyz
========================

能力Eval:
  create-user:     PASS (pass@1)
  validate-email:  PASS (pass@2)
  hash-password:   PASS (pass@1)
  全体:            3/3 成功

回帰Eval:
  login-flow:      PASS
  session-mgmt:    PASS
  logout-flow:     PASS
  全体:            3/3 成功

メトリクス:
  pass@1: 67% (2/3)
  pass@3: 100% (3/3)

ステータス: レビュー準備完了
```

## 統合パターン

### 実装前
```
/eval define feature-name
```
`.claude/evals/feature-name.md` にeval定義ファイルを作成

### 実装中
```
/eval check feature-name
```
現在のevalを実行してステータスを報告

### 実装後
```
/eval report feature-name
```
完全なevalレポートを生成

## Eval ストレージ

プロジェクトにevalを保存:
```
.claude/
  evals/
    feature-xyz.md      # Eval定義
    feature-xyz.log     # Evalの実行履歴
    baseline.json       # 回帰ベースライン
```

## ベストプラクティス

1. **コードの前にevalを定義** - 成功基準について明確に考える
2. **頻繁にevalを実行** - 早期に回帰をキャッチ
3. **時間とともにpass@kを追跡** - 信頼性の傾向を監視
4. **可能な限りコードグレーダーを使用** - 確定的 > 確率的
5. **セキュリティレビューは自動化しない** - セキュリティチェックは常に手動で
6. **evalを高速に保つ** - スローevalは実行されない
7. **evalをコードと一緒にバージョン管理** - evalは第一級の成果物

## 例: 認証を追加

```markdown
## EVAL: add-authentication

### フェーズ1: 定義（10分）
能力Eval:
- [ ] ユーザーがメール/パスワードで登録できる
- [ ] ユーザーが有効な認証情報でログインできる
- [ ] 無効な認証情報は適切なエラーで拒否
- [ ] セッションはページ再読み込みで保持
- [ ] ログアウトはセッションをクリア

回帰Eval:
- [ ] 公開ルートはまだアクセス可能
- [ ] API レスポンスは変わっていない
- [ ] データベーススキーマは互換性がある

### フェーズ2: 実装
[コードを書く]

### フェーズ3: 評価
実行: /eval check add-authentication

### フェーズ4: レポート
EVAL REPORT: add-authentication
==============================
能力: 5/5 成功 (pass@3: 100%)
回帰: 3/3 成功 (pass^3: 100%)
ステータス: SHIP IT
```
