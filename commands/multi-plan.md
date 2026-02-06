# プラン - マルチモデル協力計画

マルチモデル協力計画 - コンテキスト取得 + デュアルモデル分析 → ステップバイステップ実装計画を生成。

$ARGUMENTS

---

## コアプロトコル

- **言語プロトコル**: ツール/モデルと相互作用するときは **英語** を使用し、ユーザーとは彼らの言語でコミュニケーション
- **必須並列**: Codex/Gemini 呼び出しは `run_in_background: true` を使用する必要があります (メインスレッドをブロックしないために単一モデル呼び出しを含む)
- **コード主権**: 外部モデルは **ゼロのファイルシステム書き込みアクセス** を持つ、すべての変更は Claude
- **ロスカットメカニズム**: 現在のフェーズ出力が検証されるまで次のフェーズに進まない
- **計画のみ**: このコマンドはコンテキストを読むことと `.claude/plan/*` 計画ファイルに書くことを許可します、しかし **本番コードは決して変更しない**

---

## Multi-Model Call Specification

**Call Syntax** (parallel: use `run_in_background: true`):

```
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement>
Context: <retrieved project context>
</TASK>
OUTPUT: Step-by-step implementation plan with pseudo-code. DO NOT modify any files.
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

**Session Reuse**: Each call returns `SESSION_ID: xxx` (typically output by wrapper), **MUST save** for subsequent `/ccg:execute` use.

**Wait for Background Tasks** (max timeout 600000ms = 10 minutes):

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**IMPORTANT**:
- Must specify `timeout: 600000`, otherwise default 30 seconds will cause premature timeout
- If still incomplete after 10 minutes, continue polling with `TaskOutput`, **NEVER kill the process**
- If waiting is skipped due to timeout, **MUST call `AskUserQuestion` to ask user whether to continue waiting or kill task**

---

## 実行ワークフロー

**計画タスク**: $ARGUMENTS

### フェーズ 1: 完全なコンテキスト取得

`[Mode: Research]`

#### 1.1 プロンプト強化 (必ず最初に実行)

**`mcp__ace-tool__enhance_prompt` ツールを必ず呼び出し**:

```
mcp__ace-tool__enhance_prompt({
  prompt: "$ARGUMENTS",
  conversation_history: "<last 5-10 conversation turns>",
  project_root_path: "$PWD"
})
```

強化されたプロンプトを待機して、**すべての後続フェーズのために元の $ARGUMENTS を強化結果に置き換え**。

#### 1.2 コンテキスト取得

**`mcp__ace-tool__search_context` ツールを呼び出し**:

```
mcp__ace-tool__search_context({
  query: "<semantic query based on enhanced requirement>",
  project_root_path: "$PWD"
})
```

- 自然言語を使用してセマンティッククエリを構築 (Where/What/How)
- **決して仮定に基づいて回答しないこと**
- MCP が利用不可: ファイル検出とキーシンボルの場所には Glob + Grep にフォールバック

#### 1.3 完全性チェック

- 関連クラス、関数、変数の**完全な定義とシグネチャ**を取得する必要がある
- コンテキストが不十分な場合、**再帰的取得**をトリガー
- 出力を優先: エントリファイル + 行番号 + キーシンボル名; 曖昧さを解決する場合のみ最小限のコードスニペットを追加

#### 1.4 要件の調整

- 要件に曖昧さがまだある場合、**必ず**ユーザーに指導質問を出力
- 要件の境界が明確になるまで (省略なし、冗長性なし)

### フェーズ 2: マルチモデル協力分析

`[Mode: Analysis]`

#### 2.1 入力を分配

**Codex と Gemini の並列呼び出し** (`run_in_background: true`):

**元の要件** (プリセット意見なし) を両方のモデルに分配:

1. **Codex バックエンド分析**:
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/analyzer.md`
   - フォーカス: 技術的実現可能性、アーキテクチャへの影響、パフォーマンスの考慮、潜在的なリスク
   - 出力: 多角的ソリューション + メリット/デメリット分析

2. **Gemini フロントエンド分析**:
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/analyzer.md`
   - フォーカス: UI/UX への影響、ユーザーエクスペリエンス、ビジュアルデザイン
   - 出力: 多角的ソリューション + メリット/デメリット分析

`TaskOutput` で両方のモデルの完全な結果を待機。**SESSION_ID を保存** (`CODEX_SESSION` と `GEMINI_SESSION`)。

#### 2.2 相互検証

視点を統合し、最適化のために反復:

1. **コンセンサスを特定** (強い信号)
2. **分岐を特定** (重み付けが必要)
3. **補完的な強み**: バックエンドロジックは Codex に従う、フロントエンドデザインは Gemini に従う
4. **論理的推論**: ソリューション内の論理的ギャップを排除

#### 2.3 (オプションだが推奨) デュアルモデルプラン草案

Claude の合成計画における省略のリスクを減らすために、両方のモデルが「計画草案」を出力できます (**ファイルを変更することは許可されていません**):

1. **Codex プラン草案** (バックエンド権限):
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/architect.md`
   - 出力: ステップバイステップ計画 + 疑似コード (フォーカス: データフロー/エッジケース/エラーハンドリング/テスト戦略)

2. **Gemini プラン草案** (フロントエンド権限):
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/architect.md`
   - 出力: ステップバイステップ計画 + 疑似コード (フォーカス: 情報アーキテクチャ/相互作用/アクセシビリティ/ビジュアル一貫性)

`TaskOutput` で両方のモデルの完全な結果を待機し、彼らの提案の主要な違いを記録。

#### 2.4 実装計画を生成 (Claude 最終版)

両方の分析を合成し、**ステップバイステップ実装計画**を生成:

```markdown
## Implementation Plan: <Task Name>

### Task Type
- [ ] Frontend (→ Gemini)
- [ ] Backend (→ Codex)
- [ ] Fullstack (→ Parallel)

### Technical Solution
<Optimal solution synthesized from Codex + Gemini analysis>

### Implementation Steps
1. <Step 1> - Expected deliverable
2. <Step 2> - Expected deliverable
...

### Key Files
| File | Operation | Description |
|------|-----------|-------------|
| path/to/file.ts:L10-L50 | Modify | Description |

### Risks and Mitigation
| Risk | Mitigation |
|------|------------|

### SESSION_ID (for /ccg:execute use)
- CODEX_SESSION: <session_id>
- GEMINI_SESSION: <session_id>
```

### フェーズ 2 終了: 計画配信 (実行ではなく)

**`/ccg:plan` の責任がここで終了し、次のアクションを実行する必須**:

1. ユーザーに完全な実装計画を提示 (疑似コードを含む)
2. `.claude/plan/<feature-name>.md` に計画を保存 (要件から機能名を抽出、例: `user-auth`, `payment-module`)
3. **太字テキスト**で出力プロンプト (実際に保存されたファイルパスを使用する必須):

   ---
   **計画が生成され、`.claude/plan/actual-feature-name.md` に保存されました**

   **上記の計画をレビューしてください。以下のことができます:**
   - **計画を変更**: 何を調整する必要があるかを教えてください。計画を更新します
   - **計画を実行**: 次のコマンドを新しいセッションにコピー

   ```
   /ccg:execute .claude/plan/actual-feature-name.md
   ```
   ---

   **注記**: 上記の `actual-feature-name.md` は実際に保存されたファイル名に置き換え必須!

4. **現在の応答を直ちに終了** (ここで停止。追加のツール呼び出しなし。)

**絶対に禁止**:
- ユーザーに「Y/N」と尋ねて自動実行 (実行は `/ccg:execute` の責任)
- 本番コードへの書き込み操作
- `/ccg:execute` または実装アクション自動呼び出し
- ユーザーが明示的に変更をリクエストしていない場合のモデル呼び出しの継続トリガー

---

## 計画の保存

計画が完了した後、計画を保存:

- **最初の計画**: `.claude/plan/<feature-name>.md`
- **反復バージョン**: `.claude/plan/<feature-name>-v2.md`, `.claude/plan/<feature-name>-v3.md`...

計画ファイルの書き込みがユーザーに計画を提示する前に完了する必須。

---

## 計画変更フロー

ユーザーが計画の変更をリクエストする場合:

1. ユーザーのフィードバックに基づいて計画内容を調整
2. `.claude/plan/<feature-name>.md` ファイルを更新
3. 変更された計画を再度提示
4. ユーザーにレビューまたは再度実行するようにプロンプト

---

## 次のステップ

ユーザーが承認した後、**手動で**実行:

```bash
/ccg:execute .claude/plan/<feature-name>.md
```

---

## 主要なルール

1. **計画のみ、実装なし** – このコマンドはコード変更を実行しません
2. **Y/N プロンプトなし** – 計画を提示するだけで、ユーザーに次のステップを決定させる
3. **信頼ルール** – バックエンドは Codex に従う、フロントエンドは Gemini に従う
4. 外部モデルは **ゼロのファイルシステム書き込みアクセス**
5. **SESSION_ID ハンドオフ** – 計画は末尾に `CODEX_SESSION` / `GEMINI_SESSION` を含む必須 (`/ccg:execute resume <SESSION_ID>` 使用用)
