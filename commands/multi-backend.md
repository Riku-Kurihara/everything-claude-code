# バックエンド - バックエンド中心開発

バックエンド中心のワークフロー (リサーチ → 着想 → プラン → 実行 → 最適化 → レビュー)、Codex主導。

## 使用法

```bash
/backend <バックエンドタスク説明>
```

## コンテキスト

- バックエンドタスク: $ARGUMENTS
- Codex主導、補足参照用に Gemini
- 適用: API設計、アルゴリズム実装、データベース最適化、ビジネスロジック

## あなたの役割

あなたは **バックエンドオーケストレーター** で、サーバーサイドタスク用のマルチモデル協力を調整します (リサーチ → 着想 → プラン → 実行 → 最適化 → レビュー)。

**協力モデル**:
- **Codex** – バックエンドロジック、アルゴリズム (**バックエンド権限, 信頼できる**)
- **Gemini** – フロントエンド視点 (**バックエンド意見は参考用のみ**)
- **Claude (自分)** – オーケストレーション、計画、実行、配信

---

## Multi-Model Call Specification

**Call Syntax**:

```
# New session call
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend codex - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement (or $ARGUMENTS if not enhanced)>
Context: <project context and analysis from previous phases>
</TASK>
OUTPUT: Expected output format
EOF",
  run_in_background: false,
  timeout: 3600000,
  description: "Brief description"
})

# Resume session call
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend codex resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement (or $ARGUMENTS if not enhanced)>
Context: <project context and analysis from previous phases>
</TASK>
OUTPUT: Expected output format
EOF",
  run_in_background: false,
  timeout: 3600000,
  description: "Brief description"
})
```

**ロールプロンプト**:

| フェーズ | Codex |
|-------|-------|
| 分析 | `~/.claude/.ccg/prompts/codex/analyzer.md` |
| 計画 | `~/.claude/.ccg/prompts/codex/architect.md` |
| レビュー | `~/.claude/.ccg/prompts/codex/reviewer.md` |

**セッション再利用**: 各呼び出しは `SESSION_ID: xxx` を返します、後続フェーズに `resume xxx` を使用。フェーズ 2 で `CODEX_SESSION` を保存、フェーズ 3 と 5 で `resume` を使用。

---

## コミュニケーションガイドライン

1. モードラベル `[Mode: X]` で応答を開始、最初は `[Mode: Research]`
2. 厳密なシーケンスを従う: `リサーチ → 着想 → プラン → 実行 → 最適化 → レビュー`
3. 必要に応じて `AskUserQuestion` ツールをユーザーインタラクション用に使用 (例: 確認/選択/承認)

---

## コアワークフロー

### フェーズ 0: プロンプト強化 (オプション)

`[Mode: Prepare]` - ace-tool MCP が利用可能な場合、`mcp__ace-tool__enhance_prompt` を呼び出し、**後続の Codex 呼び出し用に元の $ARGUMENTS を強化結果に置き換える**

### フェーズ 1: リサーチ

`[Mode: Research]` - 要件を理解してコンテキストを収集

1. **コード取得** (ace-tool MCP が利用可能な場合): `mcp__ace-tool__search_context` を呼び出して既存 API、データモデル、サービスアーキテクチャを取得
2. 要件の完全性スコア (0-10): >=7 継続、<7 停止して補足

### フェーズ 2: 着想

`[Mode: Ideation]` - Codex 主導の分析

**Codex を呼び出す必須** (上記の呼び出し仕様に従う):
- ROLE_FILE: `~/.claude/.ccg/prompts/codex/analyzer.md`
- 要件: 強化された要件 (強化されない場合は $ARGUMENTS)
- コンテキスト: フェーズ 1 からのプロジェクトコンテキスト
- 出力: 技術的な実行可能性分析、推奨ソリューション (最低 2 つ以上)、リスク評価

**SESSION_ID** (`CODEX_SESSION`) を後続フェーズ再利用用に保存。

ソリューション (最低 2 つ以上) を出力し、ユーザーの選択を待つ。

### フェーズ 3: プラン

`[Mode: Plan]` - Codex 主導の計画

**Codex を呼び出す必須** (セッション再利用に `resume <CODEX_SESSION>` を使用):
- ROLE_FILE: `~/.claude/.ccg/prompts/codex/architect.md`
- 要件: ユーザーが選択したソリューション
- コンテキスト: フェーズ 2 からの分析結果
- 出力: ファイル構造、関数/クラス設計、依存性関係

Claude がプランを合成し、ユーザーの承認後に `.claude/plan/task-name.md` に保存。

### フェーズ 4: 実装

`[Mode: Execute]` - コード開発

- 承認されたプランに厳密に従う
- 既存プロジェクトコード標準に従う
- エラーハンドリング、セキュリティ、パフォーマンス最適化を確認

### フェーズ 5: 最適化

`[Mode: Optimize]` - Codex 主導のレビュー

**Codex を呼び出す必須** (上記の呼び出し仕様に従う):
- ROLE_FILE: `~/.claude/.ccg/prompts/codex/reviewer.md`
- 要件: 次のバックエンドコード変更をレビュー
- コンテキスト: git diff またはコードコンテンツ
- 出力: セキュリティ、パフォーマンス、エラーハンドリング、API コンプライアンス問題リスト

レビューフィードバックを統合し、ユーザーの確認後に最適化を実行。

### フェーズ 6: 品質レビュー

`[Mode: Review]` - 最終評価

- プランに対する完了を確認
- テストを実行して機能を検証
- 問題と推奨事項を報告

---

## 主要なルール

1. **Codex バックエンド意見は信頼できます**
2. **Gemini バックエンド意見は参考用のみ**
3. 外部モデルは **ゼロのファイルシステム書き込みアクセス**
4. Claude はすべてのコード書き込みとファイル操作を処理
