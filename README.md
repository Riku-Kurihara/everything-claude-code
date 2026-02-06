**言語:** 日本語 | [English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](docs/zh-TW/README.md)

# Everything Claude Code

[![Stars](https://img.shields.io/github/stars/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/stargazers)
[![Forks](https://img.shields.io/github/forks/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/network/members)
[![Contributors](https://img.shields.io/github/contributors/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/graphs/contributors)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Shell](https://img.shields.io/badge/-Shell-4EAA25?logo=gnu-bash&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white)
![Python](https://img.shields.io/badge/-Python-3776AB?logo=python&logoColor=white)
![Go](https://img.shields.io/badge/-Go-00ADD8?logo=go&logoColor=white)
![Java](https://img.shields.io/badge/-Java-ED8B00?logo=openjdk&logoColor=white)
![Markdown](https://img.shields.io/badge/-Markdown-000000?logo=markdown&logoColor=white)

> **41K+ スター** | **5K+ フォーク** | **22 コントリビューター** | **6言語対応**

---

**Anthropic ハッカソン優勝者による、完全な Claude Code 設定のコレクション。**

本番環境対応のエージェント、スキル、フック、コマンド、ルール、MCP設定は、実際の製品開発で10ヶ月以上の集約的な日々の使用から進化しています。

---

## ガイド

このリポジトリはコードのみです。ガイドがすべてを説明します。

<table>
<tr>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2012378465664745795">
<img src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" alt="The Shorthand Guide to Everything Claude Code" />
</a>
</td>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2014040193557471352">
<img src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" alt="The Longform Guide to Everything Claude Code" />
</a>
</td>
</tr>
<tr>
<td align="center"><b>ショートハンドガイド</b><br/>セットアップ、基礎、哲学。<b>最初にこれを読んでください。</b></td>
<td align="center"><b>ロングフォームガイド</b><br/>トークン最適化、メモリ永続化、評価、並列化。</td>
</tr>
</table>

| トピック | 学べること |
|---------|----------|
| トークン最適化 | モデル選択、システムプロンプト削減、バックグラウンドプロセス |
| メモリ永続化 | セッション間で自動的にコンテキストを保存/読み込むフック |
| 継続的学習 | セッションから再利用可能なスキルへのパターン自動抽出 |
| 検証ループ | チェックポイント vs 継続的評価、グレーダータイプ、pass@k メトリクス |
| 並列化 | Git ワークツリー、カスケード方式、インスタンス スケーリング時期 |
| サブエージェント オーケストレーション | コンテキスト問題、反復取得パターン |

---

## 新機能

### v1.4.1 — バグ修正 (2026年2月)

- **instinct インポートのコンテンツロス修正** — `/instinct-import` 中に `parse_instinct_file()` がフロントマター後のすべてのコンテンツ(Action、Evidence、Examples セクション)を誤って削除していました。コミュニティ コントリビューター @ericcai0814 により修正 ([#148](https://github.com/affaan-m/everything-claude-code/issues/148), [#161](https://github.com/affaan-m/everything-claude-code/pull/161))

### v1.4.0 — マルチ言語ルール、インストール ウィザード & PM2 (2026年2月)

- **インタラクティブ インストール ウィザード** — 新しい `configure-ecc` スキルがマージ/上書き検出を備えたガイド付きセットアップを提供
- **PM2 & マルチエージェント オーケストレーション** — 複雑なマルチサービス ワークフロー管理用の6つの新しいコマンド (`/pm2`、`/multi-plan`、`/multi-execute`、`/multi-backend`、`/multi-frontend`、`/multi-workflow`)
- **マルチ言語ルール アーキテクチャ** — ルールが フラットファイルから `common/` + `typescript/` + `python/` + `golang/` ディレクトリ に再構成。必要な言語のみをインストール
- **中国語 (zh-CN) 翻訳** — すべてのエージェント、コマンド、スキル、ルールの完全翻訳 (80+ ファイル)
- **GitHub Sponsors サポート** — GitHub Sponsors でプロジェクトをスポンサー
- **強化された CONTRIBUTING.md** — 各貢献タイプ向けの詳細なPRテンプレート

### v1.3.0 — OpenCode プラグイン サポート (2026年2月)

- **完全な OpenCode 統合** — OpenCode のプラグイン システム経由でのフック サポート付き 12 エージェント、24 コマンド、16 スキル (20+ イベント タイプ)
- **3つのネイティブ カスタム ツール** — run-tests、check-coverage、security-audit
- **LLM ドキュメント** — OpenCode ドキュメント用の `llms.txt`

### v1.2.0 — 統一コマンド & スキル (2026年2月)

- **Python/Django サポート** — Django パターン、セキュリティ、TDD、検証スキル
- **Java Spring Boot スキル** — Spring Boot 向けのパターン、セキュリティ、TDD、検証
- **セッション管理** — セッション履歴用の `/sessions` コマンド
- **継続的学習 v2** — 信頼度スコアリング、インポート/エクスポート、進化を備えた Instinct ベースの学習

全チェンジログは [Releases](https://github.com/affaan-m/everything-claude-code/releases) を参照してください。

---

## 🚀 クイック スタート

2分以内でセットアップ:

### ステップ 1: プラグインをインストール

```bash
# マーケットプレイスを追加
/plugin marketplace add affaan-m/everything-claude-code

# プラグインをインストール
/plugin install everything-claude-code@everything-claude-code
```

### ステップ 2: ルールをインストール (必須)

> ⚠️ **重要:** Claude Code プラグイン システムは `rules` を自動配布できません。手動でインストールしてください:

```bash
# リポジトリをクローン
git clone https://github.com/affaan-m/everything-claude-code.git

# 共通ルール (必須) をインストール
cp -r everything-claude-code/rules/common/* ~/.claude/rules/

# 言語別ルール (スタックを選択)
cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/
cp -r everything-claude-code/rules/python/* ~/.claude/rules/
cp -r everything-claude-code/rules/golang/* ~/.claude/rules/
```

### ステップ 3: 使用開始

```bash
# コマンドを試す
/plan "ユーザー認証を追加"

# 利用可能なコマンドをチェック
/plugin list everything-claude-code@everything-claude-code
```

✨ **完了!** 15+ エージェント、30+ スキル、30+ コマンドにアクセスできるようになりました。

---

## 🌐 クロスプラットフォーム サポート

このプラグインは **Windows、macOS、Linux** を完全サポートしています。すべてのフックとスクリプトは最大の互換性を実現するため Node.js で書き直されています。

### パッケージ マネージャー 検出

プラグインは次の優先順位で希望するパッケージ マネージャー (npm、pnpm、yarn、bun) を自動検出します:

1. **環境変数**: `CLAUDE_PACKAGE_MANAGER`
2. **プロジェクト設定**: `.claude/package-manager.json`
3. **package.json**: `packageManager` フィールド
4. **ロック ファイル**: package-lock.json、yarn.lock、pnpm-lock.yaml、bun.lockb から検出
5. **グローバル設定**: `~/.claude/package-manager.json`
6. **フォールバック**: 最初に利用可能なパッケージ マネージャー

希望するパッケージ マネージャーを設定:

```bash
# 環境変数経由
export CLAUDE_PACKAGE_MANAGER=pnpm

# グローバル設定経由
node scripts/setup-package-manager.js --global pnpm

# プロジェクト設定経由
node scripts/setup-package-manager.js --project bun

# 現在の設定を検出
node scripts/setup-package-manager.js --detect
```

または Claude Code の `/setup-pm` コマンドを使用します。

---

## 📦 内容

このリポジトリは **Claude Code プラグイン** です - 直接インストールするか、コンポーネントを手動でコピーしてください。

```
everything-claude-code/
|-- .claude-plugin/   # プラグインおよびマーケットプレイス マニフェスト
|   |-- plugin.json         # プラグイン メタデータとコンポーネント パス
|   |-- marketplace.json    # /plugin marketplace add 用マーケットプレイス カタログ
|
|-- agents/           # デリゲーション用の特化したサブエージェント
|   |-- planner.md           # 機能実装計画
|   |-- architect.md         # システム設計決定
|   |-- tdd-guide.md         # テスト駆動開発
|   |-- code-reviewer.md     # 品質とセキュリティ レビュー
|   |-- security-reviewer.md # 脆弱性分析
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E テスト
|   |-- refactor-cleaner.md  # デッド コード削除
|   |-- doc-updater.md       # ドキュメント 同期
|   |-- go-reviewer.md       # Go コード レビュー
|   |-- go-build-resolver.md # Go ビルド エラー解決
|   |-- python-reviewer.md   # Python コード レビュー (NEW)
|   |-- database-reviewer.md # データベース/Supabase レビュー (NEW)
|
|-- skills/           # ワークフロー定義とドメイン知識
|   |-- coding-standards/           # 言語ベスト プラクティス
|   |-- backend-patterns/           # API、データベース、キャッシング パターン
|   |-- frontend-patterns/          # React、Next.js パターン
|   |-- continuous-learning/        # セッションからのパターン自動抽出 (ロングフォームガイド)
|   |-- continuous-learning-v2/     # 信頼度スコアリング付き Instinct ベースの学習
|   |-- iterative-retrieval/        # サブエージェント向けの段階的コンテキスト洗練
|   |-- strategic-compact/          # 手動コンパクション提案 (ロングフォームガイド)
|   |-- tdd-workflow/               # TDD 方法論
|   |-- security-review/            # セキュリティ チェックリスト
|   |-- eval-harness/               # 検証ループ評価 (ロングフォームガイド)
|   |-- verification-loop/          # 継続的検証 (ロングフォームガイド)
|   |-- golang-patterns/            # Go イディオムとベスト プラクティス
|   |-- golang-testing/             # Go テスト パターン、TDD、ベンチマーク
|   |-- django-patterns/            # Django パターン、モデル、ビュー (NEW)
|   |-- django-security/            # Django セキュリティ ベスト プラクティス (NEW)
|   |-- django-tdd/                 # Django TDD ワークフロー (NEW)
|   |-- django-verification/        # Django 検証ループ (NEW)
|   |-- python-patterns/            # Python イディオムとベスト プラクティス (NEW)
|   |-- python-testing/             # pytest を使用した Python テスト (NEW)
|   |-- springboot-patterns/        # Java Spring Boot パターン (NEW)
|   |-- springboot-security/        # Spring Boot セキュリティ (NEW)
|   |-- springboot-tdd/             # Spring Boot TDD (NEW)
|   |-- springboot-verification/    # Spring Boot 検証 (NEW)
|   |-- configure-ecc/              # インタラクティブ インストール ウィザード (NEW)
|
|-- commands/         # クイック実行用のスラッシュ コマンド
|   |-- tdd.md              # /tdd - テスト駆動開発
|   |-- plan.md             # /plan - 実装計画
|   |-- e2e.md              # /e2e - E2E テスト生成
|   |-- code-review.md      # /code-review - 品質 レビュー
|   |-- build-fix.md        # /build-fix - ビルド エラー修正
|   |-- refactor-clean.md   # /refactor-clean - デッド コード削除
|   |-- learn.md            # /learn - セッション中のパターン抽出 (ロングフォームガイド)
|   |-- checkpoint.md       # /checkpoint - 検証状態の保存 (ロングフォームガイド)
|   |-- verify.md           # /verify - 検証ループ実行 (ロングフォームガイド)
|   |-- setup-pm.md         # /setup-pm - パッケージ マネージャー設定
|   |-- go-review.md        # /go-review - Go コード レビュー (NEW)
|   |-- go-test.md          # /go-test - Go TDD ワークフロー (NEW)
|   |-- go-build.md         # /go-build - Go ビルド エラー修正 (NEW)
|   |-- skill-create.md     # /skill-create - git 履歴からスキル生成 (NEW)
|   |-- instinct-status.md  # /instinct-status - 学習した instinct を表示 (NEW)
|   |-- instinct-import.md  # /instinct-import - instinct をインポート (NEW)
|   |-- instinct-export.md  # /instinct-export - instinct をエクスポート (NEW)
|   |-- evolve.md           # /evolve - instinct をスキルにクラスター化 (NEW)
|   |-- pm2.md              # /pm2 - PM2 サービス ライフサイクル管理 (NEW)
|   |-- multi-plan.md       # /multi-plan - マルチエージェント タスク分解 (NEW)
|   |-- multi-execute.md    # /multi-execute - オーケストレーション マルチエージェント ワークフロー (NEW)
|   |-- multi-backend.md    # /multi-backend - バックエンド マルチサービス オーケストレーション (NEW)
|   |-- multi-frontend.md   # /multi-frontend - フロントエンド マルチサービス オーケストレーション (NEW)
|   |-- multi-workflow.md   # /multi-workflow - 汎用 マルチサービス ワークフロー (NEW)
|
|-- rules/            # 常に従うべきガイドライン (~/.claude/rules/ にコピー)
|   |-- README.md            # 構造概要とインストール ガイド
|   |-- common/              # 言語不問の原則
|   |   |-- coding-style.md    # 不変性、ファイル構成
|   |   |-- git-workflow.md    # コミット形式、PR プロセス
|   |   |-- testing.md         # TDD、80% カバレッジ要件
|   |   |-- performance.md     # モデル選択、コンテキスト管理
|   |   |-- patterns.md        # デザイン パターン、スケルトン プロジェクト
|   |   |-- hooks.md           # フック アーキテクチャ、TodoWrite
|   |   |-- agents.md          # サブエージェントへのデリゲーション時期
|   |   |-- security.md        # 必須セキュリティ チェック
|   |-- typescript/          # TypeScript/JavaScript 固有
|   |-- python/              # Python 固有
|   |-- golang/              # Go 固有
|
|-- hooks/            # トリガーベースの自動化
|   |-- hooks.json                # すべてのフック設定 (PreToolUse、PostToolUse、Stop など)
|   |-- memory-persistence/       # セッション ライフサイクル フック (ロングフォームガイド)
|   |-- strategic-compact/        # コンパクション提案 (ロングフォームガイド)
|
|-- scripts/          # クロスプラットフォーム Node.js スクリプト (NEW)
|   |-- lib/                     # 共有ユーティリティ
|   |   |-- utils.js             # クロスプラットフォーム ファイル/パス/システム ユーティリティ
|   |   |-- package-manager.js   # パッケージ マネージャー 検出と選択
|   |-- hooks/                   # フック実装
|   |   |-- session-start.js     # セッション開始時にコンテキストを読み込む
|   |   |-- session-end.js       # セッション終了時に状態を保存
|   |   |-- pre-compact.js       # コンパクション前状態の保存
|   |   |-- suggest-compact.js   # 戦略的コンパクション提案
|   |   |-- evaluate-session.js  # セッションからパターンを抽出
|   |-- setup-package-manager.js # インタラクティブ PM セットアップ
|
|-- tests/            # テスト スイート (NEW)
|   |-- lib/                     # ライブラリ テスト
|   |-- hooks/                   # フック テスト
|   |-- run-all.js               # すべてのテストを実行
|
|-- contexts/         # 動的システム プロンプト注入コンテキスト (ロングフォームガイド)
|   |-- dev.md              # 開発モード コンテキスト
|   |-- review.md           # コード レビュー モード コンテキスト
|   |-- research.md         # 研究/探索モード コンテキスト
|
|-- examples/         # 設定例とセッション例
|   |-- CLAUDE.md           # プロジェクト レベル設定例
|   |-- user-CLAUDE.md      # ユーザー レベル設定例
|
|-- mcp-configs/      # MCP サーバー設定
|   |-- mcp-servers.json    # GitHub、Supabase、Vercel、Railway など
|
|-- marketplace.json  # セルフホスト型マーケットプレイス設定 (/plugin marketplace add 用)
```

---

## 🛠️ エコシステム ツール

### スキル クリエーター

リポジトリから Claude Code スキルを生成する2つの方法:

#### オプション A: ローカル分析 (組み込み)

外部サービスなしでローカル分析用に `/skill-create` コマンドを使用:

```bash
/skill-create                    # 現在のリポジトリを分析
/skill-create --instincts        # 継続的学習用の instinct も生成
```

これはあなたの git 履歴をローカルで分析し、SKILL.md ファイルを生成します。

#### オプション B: GitHub App (高度)

高度な機能 (10k+ コミット、自動PR、チーム共有) 用:

[GitHub App をインストール](https://github.com/apps/skill-creator) | [ecc.tools](https://ecc.tools)

```bash
# 任意のイシュー上でコメント:
/skill-creator analyze

# または デフォルト ブランチへのプッシュで自動トリガー
```

両方のオプションが生成:
- **SKILL.md ファイル** - Claude Code で使用可能な即座に使えるスキル
- **Instinct コレクション** - 継続的学習用
- **パターン抽出** - コミット履歴から学習

### 🧠 継続的学習 v2

Instinct ベースの学習システムが自動的にあなたのパターンを学習:

```bash
/instinct-status        # 信頼度付きの学習した instinct を表示
/instinct-import <file> # 他のユーザーの instinct をインポート
/instinct-export        # あなたの instinct をエクスポートして共有
/evolve                 # 関連する instinct をスキルにクラスター化
```

完全なドキュメントは `skills/continuous-learning-v2/` を参照してください。

---

## 📋 要件

### Claude Code CLI バージョン

**最小バージョン: v2.1.0 以降**

このプラグインはプラグイン システムがフックを処理する方法の変更により Claude Code CLI v2.1.0+ が必要です。

バージョンを確認:
```bash
claude --version
```

### 重要: フック自動読み込み動作

> ⚠️ **コントリビューター向け:** `.claude-plugin/plugin.json` に `"hooks"` フィールドを追加しないでください。これはリグレッション テストで実施されます。

Claude Code v2.1+ は規約により、インストール済みプラグインから `hooks/hooks.json` を **自動的に読み込みます**。`plugin.json` で明示的に宣言するとリダイレクト エラーが発生します:

```
Duplicate hooks file detected: ./hooks/hooks.json resolves to already-loaded file
```

**履歴:** これはこのリポジトリで繰り返しの修正/復帰サイクルを引き起こしています ([#29](https://github.com/affaan-m/everything-claude-code/issues/29)、[#52](https://github.com/affaan-m/everything-claude-code/issues/52)、[#103](https://github.com/affaan-m/everything-claude-code/issues/103))。Claude Code バージョン間で動作が変更されているため、混乱が生じています。現在、リグレッション テストでこれが再導入されるのを防いでいます。

---

## 📥 インストール

### オプション 1: プラグインとしてインストール (推奨)

このリポジトリを使用する最も簡単な方法 - Claude Code プラグインとしてインストール:

```bash
# このリポジトリをマーケットプレイスとして追加
/plugin marketplace add affaan-m/everything-claude-code

# プラグインをインストール
/plugin install everything-claude-code@everything-claude-code
```

または `~/.claude/settings.json` に直接追加:

```json
{
  "extraKnownMarketplaces": {
    "everything-claude-code": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

これにより、すべてのコマンド、エージェント、スキル、フックに即座にアクセスできます。

> **注:** Claude Code プラグイン システムはプラグイン経由での `rules` 配布をサポートしていません ([上流の制限](https://code.claude.com/docs/en/plugins-reference))。ルールを手動でインストールする必要があります:
>
> ```bash
> # リポジトリをクローン
> git clone https://github.com/affaan-m/everything-claude-code.git
>
> # オプション A: ユーザー レベル ルール (すべてのプロジェクトに適用)
> cp -r everything-claude-code/rules/common/* ~/.claude/rules/
> cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/   # スタックを選択
> cp -r everything-claude-code/rules/python/* ~/.claude/rules/
> cp -r everything-claude-code/rules/golang/* ~/.claude/rules/
>
> # オプション B: プロジェクト レベル ルール (現在のプロジェクトのみに適用)
> mkdir -p .claude/rules
> cp -r everything-claude-code/rules/common/* .claude/rules/
> cp -r everything-claude-code/rules/typescript/* .claude/rules/     # スタックを選択
> ```

---

### 🔧 オプション 2: 手動インストール

インストール対象を細かく制御したい場合:

```bash
# リポジトリをクローン
git clone https://github.com/affaan-m/everything-claude-code.git

# エージェントを Claude 設定にコピー
cp everything-claude-code/agents/*.md ~/.claude/agents/

# ルール (共通 + 言語別) をコピー
cp -r everything-claude-code/rules/common/* ~/.claude/rules/
cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/   # スタックを選択
cp -r everything-claude-code/rules/python/* ~/.claude/rules/
cp -r everything-claude-code/rules/golang/* ~/.claude/rules/

# コマンドをコピー
cp everything-claude-code/commands/*.md ~/.claude/commands/

# スキルをコピー
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

#### フックを settings.json に追加

`hooks/hooks.json` からフックをコピーして、`~/.claude/settings.json` に追加します。

#### MCP を設定

`mcp-configs/mcp-servers.json` から希望する MCP サーバーをコピーして、`~/.claude.json` に追加します。

**重要:** `YOUR_*_HERE` プレースホルダーを実際の API キーに置き換えてください。

---

## 🎯 主要概念

### エージェント

サブエージェントは限定的なスコープで委譲タスクを処理します。例:

```markdown
---
name: code-reviewer
description: コード品質、セキュリティ、保守性をレビュー
tools: ["Read", "Grep", "Glob", "Bash"]
model: opus
---

シニア コード レビュアーです...
```

### スキル

スキルはコマンドまたはエージェントから呼び出されるワークフロー定義です:

```markdown
# TDD ワークフロー

1. インターフェースを最初に定義
2. 失敗テストを作成 (RED)
3. 最小限のコードを実装 (GREEN)
4. リファクタリング (IMPROVE)
5. 80%+ カバレッジを検証
```

### フック

フックはツール イベント時に実行されます。例 - console.log について警告:

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\\\.log' \"$file_path\" && echo '[Hook] console.log を削除してください' >&2"
  }]
}
```

### ルール

ルールは常に従うべきガイドラインで、`common/` (言語不問) + 言語別ディレクトリに構成されています:

```
rules/
  common/          # 普遍的な原則 (常にインストール)
  typescript/      # TS/JS 固有パターンとツール
  python/          # Python 固有パターンとツール
  golang/          # Go 固有パターンとツール
```

詳細なインストールと構造については [`rules/README.md`](rules/README.md) を参照してください。

---

## 🧪 テストを実行

プラグインには包括的なテスト スイートが含まれています:

```bash
# すべてのテストを実行
node tests/run-all.js

# 個別のテスト ファイルを実行
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

---

## 🤝 コントリビューション

**コントリビューションは歓迎され、奨励されます。**

このリポジトリはコミュニティ リソースとなることを目的としています。以下がある場合:
- 役立つエージェントまたはスキル
- 巧妙なフック
- より良い MCP 設定
- 改善されたルール

ぜひコントリビューションしてください! ガイドラインは [CONTRIBUTING.md](CONTRIBUTING.md) を参照してください。

### コントリビューション アイデア

- 言語別スキル (Rust、C#、Swift、Kotlin) — Go、Python、Java は既に含まれています
- フレームワーク別設定 (Rails、Laravel、FastAPI、NestJS) — Django、Spring Boot は既に含まれています
- DevOps エージェント (Kubernetes、Terraform、AWS、Docker)
- テスト戦略 (異なるフレームワーク、ビジュアル リグレッション)
- ドメイン別知識 (ML、データ エンジニアリング、モバイル)

---

## 🔌 OpenCode サポート

ECC はプラグインとフックを含む **完全な OpenCode サポート** を提供します。

### クイック スタート

```bash
# OpenCode をインストール
npm install -g opencode

# リポジトリ ルートで実行
opencode
```

設定は `.opencode/opencode.json` から自動的に検出されます。

### 機能比較

| 機能 | Claude Code | OpenCode | ステータス |
|------|-------------|----------|---------|
| エージェント | ✅ 14 エージェント | ✅ 12 エージェント | **Claude Code がリード** |
| コマンド | ✅ 30 コマンド | ✅ 24 コマンド | **Claude Code がリード** |
| スキル | ✅ 28 スキル | ✅ 16 スキル | **Claude Code がリード** |
| フック | ✅ 3 フェーズ | ✅ 20+ イベント | **OpenCode がより多い!** |
| ルール | ✅ 8 ルール | ✅ 8 ルール | **完全なパリティ** |
| MCP サーバー | ✅ フル | ✅ フル | **完全なパリティ** |
| カスタム ツール | ✅ フック経由 | ✅ ネイティブ サポート | **OpenCode がより優れている** |

### プラグイン経由のフック サポート

OpenCode のプラグイン システムは Claude Code より高度な 20+ イベント タイプを備えています:

| Claude Code フック | OpenCode プラグイン イベント |
|------------------|--------------------------|
| PreToolUse | `tool.execute.before` |
| PostToolUse | `tool.execute.after` |
| Stop | `session.idle` |
| SessionStart | `session.created` |
| SessionEnd | `session.deleted` |

**追加の OpenCode イベント**: `file.edited`、`file.watcher.updated`、`message.updated`、`lsp.client.diagnostics`、`tui.toast.show` など。

### 利用可能なコマンド (24)

| コマンド | 説明 |
|---------|------|
| `/plan` | 実装計画を作成 |
| `/tdd` | TDD ワークフローを実施 |
| `/code-review` | コード変更をレビュー |
| `/security` | セキュリティ レビューを実行 |
| `/build-fix` | ビルド エラーを修正 |
| `/e2e` | E2E テストを生成 |
| `/refactor-clean` | デッド コードを削除 |
| `/orchestrate` | マルチエージェント ワークフロー |
| `/learn` | セッションからパターンを抽出 |
| `/checkpoint` | 検証状態を保存 |
| `/verify` | 検証ループを実行 |
| `/eval` | 基準に対して評価 |
| `/update-docs` | ドキュメントを更新 |
| `/update-codemaps` | コードマップを更新 |
| `/test-coverage` | カバレッジを分析 |
| `/go-review` | Go コード レビュー |
| `/go-test` | Go TDD ワークフロー |
| `/go-build` | Go ビルド エラーを修正 |
| `/skill-create` | git からスキルを生成 |
| `/instinct-status` | 学習した instinct を表示 |
| `/instinct-import` | instinct をインポート |
| `/instinct-export` | instinct をエクスポート |
| `/evolve` | instinct をスキルにクラスター化 |
| `/setup-pm` | パッケージ マネージャーを設定 |

### プラグイン インストール

**オプション 1: 直接使用**
```bash
cd everything-claude-code
opencode
```

**オプション 2: npm パッケージとしてインストール**
```bash
npm install opencode-ecc
```

`opencode.json` に追加:
```json
{
  "plugin": ["opencode-ecc"]
}
```

### ドキュメント

- **移行ガイド**: `.opencode/MIGRATION.md`
- **OpenCode プラグイン README**: `.opencode/README.md`
- **統合ルール**: `.opencode/instructions/INSTRUCTIONS.md`
- **LLM ドキュメント**: `llms.txt` (OpenCode ドキュメント完全版)

---

## 📖 背景

実験的ロールアウト以来 Claude Code を使用しています。2025年9月に Anthropic x Forum Ventures ハッカソンで [@DRodriguezFX](https://x.com/DRodriguezFX) と共に [zenith.chat](https://zenith.chat) を Claude Code のみを使用して構築して優勝しました。

これらの設定は複数の本番アプリケーション全体でテストされています。

---

## ⚠️ 重要な注意事項

### コンテキスト ウィンドウ 管理

**重要:** すべての MCP を一度に有効にしないでください。多数のツールを有効にすると、200k コンテキスト ウィンドウが 70k に縮小する可能性があります。

経験則:
- 20~30 個の MCP を設定
- プロジェクトごとに 10 個以下を有効
- 80 個以下のアクティブ ツール

プロジェクト設定で `disabledMcpServers` を使用して未使用のものを無効化します。

### カスタマイズ

これらの設定は私のワークフローに対応しています。あなたはすべき:
1. 共感するもので開始
2. スタック用に変更
3. 使用しないものを削除
4. 独自のパターンを追加

---

## 🌟 スター 履歴

[![Star History Chart](https://api.star-history.com/svg?repos=affaan-m/everything-claude-code&type=Date)](https://star-history.com/#affaan-m/everything-claude-code&Date)

---

## 🔗 リンク

- **ショートハンド ガイド (ここから開始):** [The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)
- **ロングフォーム ガイド (高度):** [The Longform Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2014040193557471352)
- **フォロー:** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat:** [zenith.chat](https://zenith.chat)

---

## 📄 ライセンス

MIT - 自由に使用、必要に応じて変更、できれば貢献してください。

---

**このリポジトリがお役に立つなら、スターしてください。両方のガイドを読んでください。素晴らしいものを構築してください。**
