---
name: continuous-learning
description: Claude Codeセッションから再利用可能なパターンを自動抽出し、将来の使用のために学習スキルとして保存します。
---

# 継続的学習スキル

Claude Codeセッション終了時に自動的に再利用可能なパターンを抽出するスキル。

## 動作方法

このスキルは**Stop フック**としてセッション末尾で実行:

1. **セッション評価**: セッションに十分なメッセージがあるか確認(デフォルト: 10以上)
2. **パターン検出**: セッションから抽出可能なパターンを特定
3. **スキル抽出**: 有用なパターンを`~/.claude/skills/learned/`に保存

## 設定

`config.json`を編集してカスタマイズ:

```json
{
  "min_session_length": 10,
  "extraction_threshold": "medium",
  "auto_approve": false,
  "learned_skills_path": "~/.claude/skills/learned/",
  "patterns_to_detect": [
    "error_resolution",
    "user_corrections",
    "workarounds",
    "debugging_techniques",
    "project_specific"
  ],
  "ignore_patterns": [
    "simple_typos",
    "one_time_fixes",
    "external_api_issues"
  ]
}
```

## パターンタイプ

| パターン | 説明 |
|---------|------|
| `error_resolution` | 特定のエラーが解決された方法 |
| `user_corrections` | ユーザー修正からのパターン |
| `workarounds` | フレームワーク/ライブラリの奇癖への解決策 |
| `debugging_techniques` | 効果的なデバッグアプローチ |
| `project_specific` | プロジェクト固有の慣例 |

## フック設定

`~/.claude/settings.json`に追加:

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning/evaluate-session.sh"
      }]
    }]
  }
}
```

## なぜStop フック?

- **軽量**: セッション終了時に1回実行
- **ノンブロッキング**: メッセージごとにレイテンシを追加しない
- **完全なコンテキスト**: セッション全体のトランスクリプトにアクセス可能

## 関連項目

- [長形式ガイド](https://x.com/affaanmustafa/status/2014040193557471352) - 継続的学習に関するセクション
- `/learn`コマンド - セッション中の手動パターン抽出

---

## 比較ノート(研究: 2025年1月)

### vs Homunculus (github.com/humanplane/homunculus)

Homunculus v2はより洗練されたアプローチを採用:

| 機能 | 我々のアプローチ | Homunculus v2 |
|------|---------|---------|
| 観察 | Stop フック(セッション末尾) | PreToolUse/PostToolUse フック(100%信頼できる) |
| 分析 | メインコンテキスト | バックグラウンドエージェント(Haiku) |
| 粒度 | スキル全体 | アトミック「本能」 |
| 信頼度 | なし | 0.3-0.9 加重 |
| 進化 | スキルに直接 | 本能 → クラスタ → スキル/コマンド/エージェント |
| 共有 | なし | 本能のエクスポート/インポート |

**Homunculusからの重要な洞察:**
> "v1はスキルに依存して観察。スキルは確率的で～50-80%の時間発動。v2は観察用にフック(100%信頼できる)を使用し、本能を学習行動のアトミックユニットとして使用。"

### v2潜在的強化

1. **本能ベース学習** - 信頼度スコアリング付きより小さなアトミック動作
2. **バックグラウンド観察者** - Haikuエージェントが並列で分析
3. **信頼度減衰** - 矛盾時に信頼度を失う本能
4. **ドメインタグ付け** - code-style、testing、git、debugging など
5. **進化パターン** - 関連本能をスキル/コマンドにクラスタ化

参照: `/Users/affoon/Documents/tasks/12-continuous-learning-v2.md` 完全仕様を参照。
