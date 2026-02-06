---
name: instinct-status
description: 信頼度レベルとともに、すべての学習済みインスティンクトを表示します
command: true
---

# インスティンクトステータスコマンド

信頼度スコア付きのすべての学習済みインスティンクトをドメインごとにグループ化して表示します。

## 実装

プラグインルートパスを使用してインスティンクト CLI を実行:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" status
```

または `CLAUDE_PLUGIN_ROOT` が設定されていない場合 (手動インストール), 使用:

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py status
```

## 使用法

```
/instinct-status
/instinct-status --domain code-style
/instinct-status --low-confidence
```

## やること

1. `~/.claude/homunculus/instincts/personal/` からすべてのインスティンクトファイルを読み込む
2. `~/.claude/homunculus/instincts/inherited/` から継承インスティンクトを読み込む
3. 信頼度バー付きでドメイン別にグループ化して表示

## 出力形式

```
📊 インスティンクトステータス
==================

## コードスタイル (4 インスティンクト)

### prefer-functional-style
トリガー: 新しい関数を書くとき
アクション: クラスより関数型パターンを使用
信頼度: ████████░░ 80%
ソース: session-observation | 最終更新: 2025-01-22

### use-path-aliases
トリガー: モジュールをインポートするとき
アクション: 相対インポートの代わりに @/ パスエイリアスを使用
信頼度: ██████░░░░ 60%
ソース: repo-analysis (github.com/acme/webapp)

## テスト (2 インスティンクト)

### test-first-workflow
トリガー: 新しい機能を追加するとき
アクション: テストを最初に書いて、その後実装
信頼度: █████████░ 90%
ソース: session-observation

## ワークフロー (3 インスティンクト)

### grep-before-edit
トリガー: コードを変更するとき
アクション: Grep で検索、Read で確認、その後 Edit
信頼度: ███████░░░ 70%
ソース: session-observation

---
合計: 9 インスティンクト (4 個人用, 5 継承)
オブザーバー: 実行中 (最後の分析: 5 分前)
```

## フラグ

- `--domain <name>`: ドメイン別にフィルタリング (code-style, testing, git など)
- `--low-confidence`: 信頼度 < 0.5 のインスティンクトのみを表示
- `--high-confidence`: 信頼度 >= 0.7 のインスティンクトのみを表示
- `--source <type>`: ソース別にフィルタリング (session-observation, repo-analysis, inherited)
- `--json`: プログラム的な使用のために JSON として出力
