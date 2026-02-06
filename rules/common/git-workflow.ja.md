# Git ワークフロー

## コミットメッセージ形式

```
<type>: <description>

<optional body>
```

種類: feat, fix, refactor, docs, test, chore, perf, ci

注：属性情報は ~/.claude/settings.json でグローバルに無効化されています。

## プルリクエストワークフロー

PR を作成するときは：
1. 完全なコミット履歴を分析（最新のコミットだけではなく）
2. `git diff [base-branch]...HEAD` を使用してすべての変更を確認
3. 包括的な PR サマリーを作成
4. TODO を含むテスト計画を含める
5. 新しいブランチの場合は `-u` フラグで push

## 機能実装ワークフロー

1. **最初に計画**
   - **planner** エージェントを使用して実装計画を作成
   - 依存関係とリスクを特定
   - フェーズに分割

2. **TDD アプローチ**
   - **tdd-guide** エージェントを使用
   - 最初にテストを書く（RED）
   - テストをパスするように実装（GREEN）
   - リファクタリング（IMPROVE）
   - 80%以上のカバレッジを確認

3. **コードレビュー**
   - コードを書いたらすぐに **code-reviewer** エージェントを使用
   - CRITICAL と HIGH の問題に対応
   - 可能な限り MEDIUM の問題を修正

4. **コミット & Push**
   - 詳細なコミットメッセージ
   - Conventional Commits 形式に従う
