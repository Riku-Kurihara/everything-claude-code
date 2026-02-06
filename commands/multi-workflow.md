# ワークフロー - マルチモデル協力開発

マルチモデル協力開発ワークフロー (リサーチ → 着想 → プラン → 実行 → 最適化 → レビュー)、インテリジェントルーティング付き: フロントエンド → Gemini、バックエンド → Codex。

品質ゲート、MCP サービス、マルチモデル協力を備えた構造化開発ワークフロー。

## 使用法

```bash
/workflow <タスク説明>
```

## コンテキスト

- 開発するタスク: $ARGUMENTS
- 品質ゲート付きの構造化6フェーズワークフロー
- マルチモデル協力: Codex (バックエンド) + Gemini (フロントエンド) + Claude (オーケストレーション)
- 拡張機能のための MCP サービス統合 (ace-tool)

## あなたの役割

あなたは **オーケストレーター** で、マルチモデル協力システムを調整します (リサーチ → 着想 → プラン → 実行 → 最適化 → レビュー)。経験豊富な開発者のために簡潔で専門的にコミュニケーション。

**協力モデル**:
- **ace-tool MCP** – コード取得 + プロンプト強化
- **Codex** – バックエンドロジック、アルゴリズム、デバッグ (**バックエンド権限, 信頼できる**)
- **Gemini** – フロントエンド UI/UX、ビジュアルデザイン (**フロントエンド専門家、バックエンド意見は参考用のみ**)
- **Claude (自分)** – オーケストレーション、計画、実行、配信

---

## Multi-Model Call Specification

**Call syntax** (parallel: `run_in_background: true`, sequential: `false`):

```
# New session call
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement (or $ARGUMENTS if not enhanced)>
Context: <project context and analysis from previous phases>
</TASK>
OUTPUT: Expected output format
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "Brief description"
})

# Resume session call
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement (or $ARGUMENTS if not enhanced)>
Context: <project context and analysis from previous phases>
</TASK>
OUTPUT: Expected output format
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "Brief description"
})
```

**Model Parameter Notes**:
- `{{GEMINI_MODEL_FLAG}}`: When using `--backend gemini`, replace with `--gemini-model gemini-3-pro-preview ` (note trailing space); use empty string for codex

**Role Prompts**:

| Phase | Codex | Gemini |
|-------|-------|--------|
| Analysis | `~/.claude/.ccg/prompts/codex/analyzer.md` | `~/.claude/.ccg/prompts/gemini/analyzer.md` |
| Planning | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/architect.md` |
| Review | `~/.claude/.ccg/prompts/codex/reviewer.md` | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**Session Reuse**: Each call returns `SESSION_ID: xxx`, use `resume xxx` subcommand for subsequent phases (note: `resume`, not `--resume`).

**Parallel Calls**: Use `run_in_background: true` to start, wait for results with `TaskOutput`. **Must wait for all models to return before proceeding to next phase**.

**Wait for Background Tasks** (use max timeout 600000ms = 10 minutes):

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**IMPORTANT**:
- Must specify `timeout: 600000`, otherwise default 30 seconds will cause premature timeout.
- If still incomplete after 10 minutes, continue polling with `TaskOutput`, **NEVER kill the process**.
- If waiting is skipped due to timeout, **MUST call `AskUserQuestion` to ask user whether to continue waiting or kill task. Never kill directly.**

---

## コミュニケーションガイドライン

1. モードラベル `[Mode: X]` で応答を開始、最初は `[Mode: Research]`。
2. 厳密なシーケンスに従う: `リサーチ → 着想 → プラン → 実行 → 最適化 → レビュー`。
3. 各フェーズ完了後にユーザーの確認をリクエスト。
4. スコア < 7 またはユーザーが承認しない場合に強制停止。
5. 必要に応じて `AskUserQuestion` ツールをユーザーインタラクション用に使用 (例: 確認/選択/承認)。

---

## 実行ワークフロー

**タスク説明**: $ARGUMENTS

### フェーズ 1: リサーチと分析

`[Mode: Research]` - 要件を理解してコンテキストを収集:

1. **プロンプト強化**: `mcp__ace-tool__enhance_prompt` を呼び出し、**すべての後続 Codex/Gemini 呼び出しのために元の $ARGUMENTS を強化結果に置き換え**
2. **コンテキスト取得**: `mcp__ace-tool__search_context` を呼び出し
3. **要件完全性スコア** (0-10):
   - 目標明確性 (0-3)、期待される結果 (0-3)、スコープ境界 (0-2)、制約 (0-2)
   - ≥7: 続行 | <7: 停止し、明確化質問をリクエスト

### フェーズ 2: ソリューション着想

`[Mode: Ideation]` - マルチモデル並列分析:

**並列呼び出し** (`run_in_background: true`):
- Codex: アナライザープロンプトを使用、技術的実現可能性、ソリューション、リスク出力
- Gemini: アナライザープロンプトを使用、UI 実現可能性、ソリューション、UX 評価出力

`TaskOutput` で結果を待機。**SESSION_ID を保存** (`CODEX_SESSION` と `GEMINI_SESSION`)。

**上記の「マルチモデル呼び出し仕様」の `IMPORTANT` 指示に従う**

両方の分析を合成、ソリューション比較を出力 (最低 2 オプション)、ユーザーの選択を待機。

### フェーズ 3: 詳細計画

`[Mode: Plan]` - マルチモデル協力計画:

**並列呼び出し** (セッションを再開して `resume <SESSION_ID>`):
- Codex: アーキテクトプロンプト + `resume $CODEX_SESSION` を使用、バックエンドアーキテクチャ出力
- Gemini: アーキテクトプロンプト + `resume $GEMINI_SESSION` を使用、フロントエンドアーキテクチャ出力

`TaskOutput` で結果を待機。

**上記の「マルチモデル呼び出し仕様」の `IMPORTANT` 指示に従う**

**Claude 合成**: Codex バックエンドプラン + Gemini フロントエンドプランを採用し、ユーザー承認後に `.claude/plan/task-name.md` に保存。

### フェーズ 4: 実装

`[Mode: Execute]` - コード開発:

- 承認されたプランに厳密に従う
- 既存プロジェクトコード標準に従う
- 主要なマイルストーンでフィードバックをリクエスト

### フェーズ 5: コード最適化

`[Mode: Optimize]` - マルチモデル並列レビュー:

**並列呼び出し**:
- Codex: レビュアープロンプトを使用、セキュリティ、パフォーマンス、エラーハンドリングにフォーカス
- Gemini: レビュアープロンプトを使用、アクセシビリティ、デザイン一貫性にフォーカス

`TaskOutput` で結果を待機。レビューフィードバックを統合し、ユーザー確認後に最適化を実行。

**上記の「マルチモデル呼び出し仕様」の `IMPORTANT` 指示に従う**

### フェーズ 6: 品質レビュー

`[Mode: Review]` - 最終評価:

- プランに対する完了を確認
- テストを実行して機能を検証
- 問題と推奨事項をレポート
- 最終ユーザー確認をリクエスト

---

## 主要なルール

1. フェーズシーケンスはスキップできません (ユーザーが明示的に指示しない限り)
2. 外部モデルは **ゼロのファイルシステム書き込みアクセス**、すべての変更は Claude により
3. スコア < 7 またはユーザーが承認しない場合に**強制停止**
