# 実行 - マルチモデル協力実行

マルチモデル協力実行 - プランからプロトタイプを取得 → Claude がリファクタリングして実装 → マルチモデル監査と配信。

$ARGUMENTS

---

## コアプロトコル

- **言語プロトコル**: ツール/モデルと相互作用するときは **英語** を使用し、ユーザーとは彼らの言語でコミュニケーション
- **コード主権**: 外部モデルは **ゼロのファイルシステム書き込みアクセス** を持つ、すべての変更は Claude
- **ダーティプロトタイプリファクタリング**: Codex/Gemini 統一Diff を「ダーティプロトタイプ」として扱う、本番品質のコードにリファクタリング必須
- **ロスカットメカニズム**: 現在のフェーズ出力が検証されるまで次のフェーズに進まない
- **前提条件**: ユーザーが `/ccg:plan` 出力に明示的に「Y」で返信した後のみ実行 (不足している場合、まず確認する必要があります)

---

## マルチモデル呼び出し仕様

**呼び出し構文** (並列: `run_in_background: true` を使用):

```
# セッション再開呼び出し (推奨) - 実装プロトタイプ
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <ロールプロンプトパス>
<TASK>
要件: <タスク説明>
コンテキスト: <プラン内容 + ターゲットファイル>
</TASK>
OUTPUT: 統一 Diff パッチのみ。実際の変更を厳密に禁止します。
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "簡潔な説明"
})

# 新規セッション呼び出し - 実装プロトタイプ
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <ロールプロンプトパス>
<TASK>
要件: <タスク説明>
コンテキスト: <プラン内容 + ターゲットファイル>
</TASK>
OUTPUT: 統一 Diff パッチのみ。実際の変更を厳密に禁止します。
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "簡潔な説明"
})
```

**監査呼び出し構文** (コードレビュー / 監査):

```
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <ロールプロンプトパス>
<TASK>
スコープ: 最終的なコード変更を監査します。
入力:
- 適用されたパッチ (git diff / 最終統一 diff)
- 触れたファイル (必要に応じて関連する抜粋)
制約:
- ファイルを変更しないでください。
- ファイルシステムアクセスを想定するツールコマンドを出力しないでください。
</TASK>
OUTPUT:
1) 優先順位付きの問題リスト (重要度、ファイル、根拠)
2) 具体的な修正。コード変更が必要な場合は、フェンス付きコードブロックに統一 Diff パッチを含めます。
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "簡潔な説明"
})
```

**モデルパラメーター注記**:
- `{{GEMINI_MODEL_FLAG}}`: `--backend gemini` を使用する場合、`--gemini-model gemini-3-pro-preview ` (末尾のスペースに注意) に置き換える; codex の場合は空文字列を使用

**ロールプロンプト**:

| フェーズ | Codex | Gemini |
|-------|-------|--------|
| 実装 | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/frontend.md` |
| レビュー | `~/.claude/.ccg/prompts/codex/reviewer.md` | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**セッション再利用**: `/ccg:plan` が SESSION_ID を提供した場合、`resume <SESSION_ID>` を使用してコンテキストを再利用します。

**バックグラウンドタスク待機** (最大タイムアウト 600000ms = 10 分):

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**重要**:
- `timeout: 600000` を指定する必要があります。そうでないとデフォルト 30 秒でタイムアウトが早すぎます
- 10 分後もまだ完了しない場合、`TaskOutput` でのポーリングを続行します。**プロセスを終了しないでください**
- タイムアウトのため待機がスキップされた場合、**ユーザーに続行するか タスクを終了するか尋ねるために `AskUserQuestion` を呼び出す必須**

---

## Execution Workflow

**Execute Task**: $ARGUMENTS

### Phase 0: Read Plan

`[Mode: Prepare]`

1. **Identify Input Type**:
   - Plan file path (e.g., `.claude/plan/xxx.md`)
   - Direct task description

2. **Read Plan Content**:
   - If plan file path provided, read and parse
   - Extract: task type, implementation steps, key files, SESSION_ID

3. **Pre-Execution Confirmation**:
   - If input is "direct task description" or plan missing `SESSION_ID` / key files: confirm with user first
   - If cannot confirm user replied "Y" to plan: must confirm again before proceeding

4. **Task Type Routing**:

   | Task Type | Detection | Route |
   |-----------|-----------|-------|
   | **Frontend** | Pages, components, UI, styles, layout | Gemini |
   | **Backend** | API, interfaces, database, logic, algorithms | Codex |
   | **Fullstack** | Contains both frontend and backend | Codex ∥ Gemini parallel |

---

### Phase 1: Quick Context Retrieval

`[Mode: Retrieval]`

**Must use MCP tool for quick context retrieval, do NOT manually read files one by one**

Based on "Key Files" list in plan, call `mcp__ace-tool__search_context`:

```
mcp__ace-tool__search_context({
  query: "<semantic query based on plan content, including key files, modules, function names>",
  project_root_path: "$PWD"
})
```

**Retrieval Strategy**:
- Extract target paths from plan's "Key Files" table
- Build semantic query covering: entry files, dependency modules, related type definitions
- If results insufficient, add 1-2 recursive retrievals
- **NEVER** use Bash + find/ls to manually explore project structure

**After Retrieval**:
- Organize retrieved code snippets
- Confirm complete context for implementation
- Proceed to Phase 3

---

### Phase 3: Prototype Acquisition

`[Mode: Prototype]`

**Route Based on Task Type**:

#### Route A: Frontend/UI/Styles → Gemini

**Limit**: Context < 32k tokens

1. Call Gemini (use `~/.claude/.ccg/prompts/gemini/frontend.md`)
2. Input: Plan content + retrieved context + target files
3. OUTPUT: `Unified Diff Patch ONLY. Strictly prohibit any actual modifications.`
4. **Gemini is frontend design authority, its CSS/React/Vue prototype is the final visual baseline**
5. **WARNING**: Ignore Gemini's backend logic suggestions
6. If plan contains `GEMINI_SESSION`: prefer `resume <GEMINI_SESSION>`

#### Route B: Backend/Logic/Algorithms → Codex

1. Call Codex (use `~/.claude/.ccg/prompts/codex/architect.md`)
2. Input: Plan content + retrieved context + target files
3. OUTPUT: `Unified Diff Patch ONLY. Strictly prohibit any actual modifications.`
4. **Codex is backend logic authority, leverage its logical reasoning and debug capabilities**
5. If plan contains `CODEX_SESSION`: prefer `resume <CODEX_SESSION>`

#### Route C: Fullstack → Parallel Calls

1. **Parallel Calls** (`run_in_background: true`):
   - Gemini: Handle frontend part
   - Codex: Handle backend part
2. Wait for both models' complete results with `TaskOutput`
3. Each uses corresponding `SESSION_ID` from plan for `resume` (create new session if missing)

**Follow the `IMPORTANT` instructions in `Multi-Model Call Specification` above**

---

### Phase 4: Code Implementation

`[Mode: Implement]`

**Claude as Code Sovereign executes the following steps**:

1. **Read Diff**: Parse Unified Diff Patch returned by Codex/Gemini

2. **Mental Sandbox**:
   - Simulate applying Diff to target files
   - Check logical consistency
   - Identify potential conflicts or side effects

3. **Refactor and Clean**:
   - Refactor "dirty prototype" to **highly readable, maintainable, enterprise-grade code**
   - Remove redundant code
   - Ensure compliance with project's existing code standards
   - **Do not generate comments/docs unless necessary**, code should be self-explanatory

4. **Minimal Scope**:
   - Changes limited to requirement scope only
   - **Mandatory review** for side effects
   - Make targeted corrections

5. **Apply Changes**:
   - Use Edit/Write tools to execute actual modifications
   - **Only modify necessary code**, never affect user's other existing functionality

6. **Self-Verification** (strongly recommended):
   - Run project's existing lint / typecheck / tests (prioritize minimal related scope)
   - If failed: fix regressions first, then proceed to Phase 5

---

### Phase 5: Audit and Delivery

`[Mode: Audit]`

#### 5.1 Automatic Audit

**After changes take effect, MUST immediately parallel call** Codex and Gemini for Code Review:

1. **Codex Review** (`run_in_background: true`):
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/reviewer.md`
   - Input: Changed Diff + target files
   - Focus: Security, performance, error handling, logic correctness

2. **Gemini Review** (`run_in_background: true`):
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/reviewer.md`
   - Input: Changed Diff + target files
   - Focus: Accessibility, design consistency, user experience

Wait for both models' complete review results with `TaskOutput`. Prefer reusing Phase 3 sessions (`resume <SESSION_ID>`) for context consistency.

#### 5.2 Integrate and Fix

1. Synthesize Codex + Gemini review feedback
2. Weigh by trust rules: Backend follows Codex, Frontend follows Gemini
3. Execute necessary fixes
4. Repeat Phase 5.1 as needed (until risk is acceptable)

#### 5.3 Delivery Confirmation

After audit passes, report to user:

```markdown
## Execution Complete

### Change Summary
| File | Operation | Description |
|------|-----------|-------------|
| path/to/file.ts | Modified | Description |

### Audit Results
- Codex: <Passed/Found N issues>
- Gemini: <Passed/Found N issues>

### Recommendations
1. [ ] <Suggested test steps>
2. [ ] <Suggested verification steps>
```

---

## Key Rules

1. **Code Sovereignty** – All file modifications by Claude, external models have zero write access
2. **Dirty Prototype Refactoring** – Codex/Gemini output treated as draft, must refactor
3. **Trust Rules** – Backend follows Codex, Frontend follows Gemini
4. **Minimal Changes** – Only modify necessary code, no side effects
5. **Mandatory Audit** – Must perform multi-model Code Review after changes

---

## Usage

```bash
# Execute plan file
/ccg:execute .claude/plan/feature-name.md

# Execute task directly (for plans already discussed in context)
/ccg:execute implement user authentication based on previous plan
```

---

## Relationship with /ccg:plan

1. `/ccg:plan` generates plan + SESSION_ID
2. User confirms with "Y"
3. `/ccg:execute` reads plan, reuses SESSION_ID, executes implementation
