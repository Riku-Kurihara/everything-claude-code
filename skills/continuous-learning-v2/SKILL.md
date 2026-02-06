---
name: continuous-learning-v2
description: フック経由でセッションを観察するインスタンスベース学習システム、信頼度スコアリング付きアトミック本能を作成し、スキル/コマンド/エージェントに進化させます。
version: 2.0.0
---

# 継続的学習 v2 - 本能ベース設計

Claude Codeセッションを再利用可能な知識に変える高度な学習システム。小さな学習行動である「本能」を通じて — 信頼度スコアリング付き。

## v2の新機能

| 機能 | v1 | v2 |
|------|----|----|
| 観察 | Stop フック(セッション末尾) | PreToolUse/PostToolUse(100%信頼できる) |
| 分析 | メインコンテキスト | バックグラウンドエージェント(Haiku) |
| 粒度 | スキル全体 | アトミック「本能」 |
| 信頼度 | なし | 0.3-0.9 加重 |
| 進化 | スキルに直接 | 本能 → クラスタ → スキル/コマンド/エージェント |
| 共有 | なし | 本能のエクスポート/インポート |

## 本能モデル

本能は小さな学習行動:

```yaml
---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.7
domain: "code-style"
source: "session-observation"
---

# 関数型スタイルを優先

## アクション
適切な場合、クラスベースのアプローチではなく関数型パターンを使用します。

## 証拠
- 関数型パターン優先の5つのインスタンスが観察されました
- ユーザーが2025-01-15にクラスベースアプローチを関数型に修正
```

**プロパティ:**
- **アトミック** — 1つのトリガー、1つのアクション
- **信頼度加重** — 0.3 = 暫定、0.9 = ほぼ確実
- **ドメインタグ付け** — code-style、testing、git、debugging、workflow など
- **証拠バック** — 何がこれを作成したかを追跡

## 動作方法

```
セッションアクティビティ
      │
      │ フックがプロンプト+ツール使用をキャプチャ(100%信頼できる)
      ▼
┌─────────────────────────────────────────┐
│         observations.jsonl              │
│   (プロンプト、ツール呼び出し、結果)     │
└─────────────────────────────────────────┘
      │
      │ 観察者エージェントが読み込み(バックグラウンド、Haiku)
      ▼
┌─────────────────────────────────────────┐
│          パターン検出                   │
│   • ユーザー修正 → 本能                 │
│   • エラー解決 → 本能                   │
│   • 繰り返しワークフロー → 本能         │
└─────────────────────────────────────────┘
      │
      │ 作成/更新
      ▼
┌─────────────────────────────────────────┐
│         instincts/personal/             │
│   • prefer-functional.md (0.7)          │
│   • always-test-first.md (0.9)          │
│   • use-zod-validation.md (0.6)         │
└─────────────────────────────────────────┘
      │
      │ /evolve クラスタ
      ▼
┌─────────────────────────────────────────┐
│              evolved/                   │
│   • commands/new-feature.md             │
│   • skills/testing-workflow.md          │
│   • agents/refactor-specialist.md       │
└─────────────────────────────────────────┘
```

## クイックスタート

### 1. 観察フックを有効化

`~/.claude/settings.json`に追加します。

**プラグインとしてインストール(推奨):**

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/hooks/observe.sh pre"
      }]
    }],
    "PostToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/hooks/observe.sh post"
      }]
    }]
  }
}
```

**手動で`~/.claude/skills`にインストール:**

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh pre"
      }]
    }],
    "PostToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh post"
      }]
    }]
  }
}
```

### 2. ディレクトリ構造を初期化

Python CLIが自動的に作成しますが、手動で作成することもできます:

```bash
mkdir -p ~/.claude/homunculus/{instincts/{personal,inherited},evolved/{agents,skills,commands}}
touch ~/.claude/homunculus/observations.jsonl
```

### 3. インスティンクトコマンドを使用

```bash
/instinct-status     # 学習した本能と信頼度スコアを表示
/evolve              # 関連本能をスキル/コマンドにクラスタ化
/instinct-export     # 本能をエクスポート(共有用)
/instinct-import     # 他者から本能をインポート
```

## コマンド

| コマンド | 説明 |
|---------|------|
| `/instinct-status` | 信頼度付きすべての学習した本能を表示 |
| `/evolve` | 関連本能をスキル/コマンドにクラスタ化 |
| `/instinct-export` | 本能をエクスポート(共有用) |
| `/instinct-import <file>` | 他者から本能をインポート |

## 設定

`config.json`を編集:

```json
{
  "version": "2.0",
  "observation": {
    "enabled": true,
    "store_path": "~/.claude/homunculus/observations.jsonl",
    "max_file_size_mb": 10,
    "archive_after_days": 7
  },
  "instincts": {
    "personal_path": "~/.claude/homunculus/instincts/personal/",
    "inherited_path": "~/.claude/homunculus/instincts/inherited/",
    "min_confidence": 0.3,
    "auto_approve_threshold": 0.7,
    "confidence_decay_rate": 0.05
  },
  "observer": {
    "enabled": true,
    "model": "haiku",
    "run_interval_minutes": 5,
    "patterns_to_detect": [
      "user_corrections",
      "error_resolutions",
      "repeated_workflows",
      "tool_preferences"
    ]
  },
  "evolution": {
    "cluster_threshold": 3,
    "evolved_path": "~/.claude/homunculus/evolved/"
  }
}
```

## ファイル構造

```
~/.claude/homunculus/
├── identity.json           # プロフィール、技術レベル
├── observations.jsonl      # 現在のセッション観察
├── observations.archive/   # 処理済み観察
├── instincts/
│   ├── personal/           # 自動学習した本能
│   └── inherited/          # 他者からインポートした本能
└── evolved/
    ├── agents/             # 生成されたスペシャリストエージェント
    ├── skills/             # 生成されたスキル
    └── commands/           # 生成されたコマンド
```

## スキルクリエイターとの統合

[スキルクリエイターGitHubアプリ](https://skill-creator.app)を使用すると、今では以下の両方が生成されます:
- 従来のSKILL.mdファイル(後方互換性向け)
- インスティンクトコレクション(v2学習システム向け)

リポジトリ分析からのインスティンクトには`source: "repo-analysis"`があり、ソースリポジトリURLを含みます。

## 信頼度スコアリング

信頼度は時間とともに進化:

| スコア | 意味 | 動作 |
|-------|------|------|
| 0.3 | 暫定 | 提案されるが強制されない |
| 0.5 | 中程度 | 関連するときに適用 |
| 0.7 | 強力 | 適用向けに自動承認 |
| 0.9 | ほぼ確実 | コア動作 |

**信頼度が上昇:**
- パターンが繰り返し観察される
- ユーザーが提案された動作を修正しない
- 他のソースからの同様の本能が同意

**信頼度が下降:**
- ユーザーが動作を明示的に修正
- パターンが長期間観察されない
- 矛盾する証拠が出現

## なぜフック経由の観察がスキルより優れているのか?

> "v1はスキルに依存して観察。スキルは確率的—Claude判断に基づき～50-80%の時間発動。"

フックは**100%の時間**確定的に発動。これは以下を意味:
- すべてのツール呼び出しが観察される
- パターンが失われない
- 学習が包括的

## 後方互換性

v2は完全にv1と互換:
- 既存の`~/.claude/skills/learned/`スキルはまだ機能
- Stop フックはまだ実行(v2にも給餌)
- 段階的な移行パス: 両方を並列で実行

## プライバシー

- 観察は**ローカル**マシンに留まる
- **パターン**のみ(インスティンクト)をエクスポート可能
- 実際のコードやコンバーセーション内容は共有されない
- 何がエクスポートされるかはユーザーが管理

## 関連項目

- [スキルクリエイター](https://skill-creator.app) - リポジトリ履歴から本能を生成
- [Homunculus](https://github.com/humanplane/homunculus) - v2設計のインスピレーション
- [長形式ガイド](https://x.com/affaanmustafa/status/2014040193557471352) - 継続的学習セクション

---

*本能ベース学習: Claude に1つの観察ずつパターンを教える。*
