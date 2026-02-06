---
name: instinct-import
description: 同僚、スキル作成者、またはその他のソースからインスティンクトをインポート
command: true
---

# インスティンクトインポートコマンド

## 実装

プラグインルートパスを使用してインスティンクト CLI を実行:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" import <file-or-url> [--dry-run] [--force] [--min-confidence 0.7]
```

または `CLAUDE_PLUGIN_ROOT` が設定されていない場合 (手動インストール):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py import <file-or-url>
```

以下からインスティンクトをインポート:
- 同僚のエクスポート
- スキル作成者 (リポジトリ分析)
- コミュニティコレクション
- 前のマシンバックアップ

## 使用法

```
/instinct-import team-instincts.yaml
/instinct-import https://github.com/org/repo/instincts.yaml
/instinct-import --from-skill-creator acme/webapp
```

## やること

1. インスティンクトファイルをフェッチ (ローカルパスまたはURL)
2. フォーマットを解析して検証
3. 既存インスティンクトとの重複をチェック
4. マージするか新しいインスティンクトを追加
5. `~/.claude/homunculus/instincts/inherited/` に保存

## インポートプロセス

```
📥 インポート元: team-instincts.yaml
================================================

インポートするインスティンクト: 12個

競合を分析中...

## 新規インスティンクト (8)
追加されます:
  ✓ use-zod-validation (信頼度: 0.7)
  ✓ prefer-named-exports (信頼度: 0.65)
  ✓ test-async-functions (信頼度: 0.8)
  ...

## 重複インスティンクト (3)
既に類似のインスティンクトがあります:
  ⚠️ prefer-functional-style
     ローカル: 0.8 信頼度, 12 回の観察
     インポート: 0.7 信頼度
     → ローカルを保持 (より高い信頼度)

  ⚠️ test-first-workflow
     ローカル: 0.75 信頼度
     インポート: 0.9 信頼度
     → インポートに更新 (より高い信頼度)

## 競合インスティンクト (1)
これらはローカルインスティンクトと矛盾:
  ❌ use-classes-for-services
     競合先: avoid-classes
     → スキップ (手動解決が必要)

---
新規 8 個を追加、1 個を更新、3 個をスキップしますか?
```

## マージ戦略

### 重複の場合
既存のインスティンクトと一致するインスティンクトをインポートするとき:
- **より高い信頼度が勝つ**: より高い信頼度のものを保持
- **証拠をマージ**: 観察数を結合
- **タイムスタンプを更新**: 最近検証されたものとしてマーク

### 競合の場合
既存のインスティンクトと矛盾するインスティンクトをインポートするとき:
- **デフォルトでスキップ**: 競合するインスティンクトをインポートしない
- **レビュー用にフラグ**: 両方を注意が必要として マーク
- **手動解決**: ユーザーが保持するものを決定

## ソース追跡

インポートされたインスティンクトは以下でマーク:
```yaml
source: "inherited"
imported_from: "team-instincts.yaml"
imported_at: "2025-01-22T10:30:00Z"
original_source: "session-observation"  # または "repo-analysis"
```

## スキルクリエーター統合

スキルクリエーターからインポートするとき:

```
/instinct-import --from-skill-creator acme/webapp
```

リポジトリ分析から生成されたインスティンクトをフェッチ:
- ソース: `repo-analysis`
- より高い初期信頼度 (0.7+)
- ソースリポジトリにリンク

## フラグ

- `--dry-run`: インポートせずにプレビュー
- `--force`: 競合が存在しても インポート
- `--merge-strategy <higher|local|import>`: 重複の処理方法
- `--from-skill-creator <owner/repo>`: スキルクリエーター分析からインポート
- `--min-confidence <n>`: 閾値以上のインスティンクトのみをインポート

## 出力

インポート後:
```
✅ インポート完了!

追加: 8 インスティンクト
更新: 1 インスティンクト
スキップ: 3 インスティンクト (重複 2、競合 1)

新規インスティンクトは以下に保存されました: ~/.claude/homunculus/instincts/inherited/

/instinct-status を実行してすべてのインスティンクトを表示してください。
```
