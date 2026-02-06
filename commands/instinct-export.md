---
name: instinct-export
description: 同僚または他のプロジェクトと共有するためにインスティンクトをエクスポートします
command: /instinct-export
---

# インスティンクトエクスポートコマンド

インスティンクトを共有可能な形式にエクスポートします。以下に最適です:
- 同僚との共有
- 新しいマシンへの転送
- プロジェクト規約への貢献

## 使用法

```
/instinct-export                           # すべての個人用インスティンクトをエクスポート
/instinct-export --domain testing          # テストインスティンクトのみをエクスポート
/instinct-export --min-confidence 0.7      # 高信頼度インスティンクトのみをエクスポート
/instinct-export --output team-instincts.yaml
```

## やること

1. `~/.claude/homunculus/instincts/personal/` からインスティンクトを読み込む
2. フラグに基づいてフィルタリング
3. 機密情報を削除:
   - セッション ID を削除
   - ファイルパスを削除 (パターンのみ保持)
   - 「先週」より古いタイムスタンプを削除
4. エクスポートファイルを生成

## 出力形式

YAML ファイルを作成:

```yaml
# インスティンクトエクスポート
# 生成日時: 2025-01-22
# ソース: personal
# 数: 12 インスティンクト

version: "2.0"
exported_by: "continuous-learning-v2"
export_date: "2025-01-22T10:30:00Z"

instincts:
  - id: prefer-functional-style
    trigger: "新しい関数を書くとき"
    action: "クラスより関数型パターンを使用"
    confidence: 0.8
    domain: code-style
    observations: 8

  - id: test-first-workflow
    trigger: "新しい機能を追加するとき"
    action: "テストを最初に書いて、その後実装"
    confidence: 0.9
    domain: testing
    observations: 12

  - id: grep-before-edit
    trigger: "コードを変更するとき"
    action: "Grep で検索、Read で確認、その後 Edit"
    confidence: 0.7
    domain: workflow
    observations: 6
```

## プライバシーに関する考慮事項

エクスポートに含まれるもの:
- ✅ トリガーパターン
- ✅ アクション
- ✅ 信頼度スコア
- ✅ ドメイン
- ✅ 観察数

エクスポートに含まれないもの:
- ❌ 実際のコードスニペット
- ❌ ファイルパス
- ❌ セッショントランスクリプト
- ❌ 個人識別子

## フラグ

- `--domain <name>`: 指定されたドメインのみをエクスポート
- `--min-confidence <n>`: 最小信頼度閾値 (デフォルト: 0.3)
- `--output <file>`: 出力ファイルパス (デフォルト: instincts-export-YYYYMMDD.yaml)
- `--format <yaml|json|md>`: 出力形式 (デフォルト: yaml)
- `--include-evidence`: 証拠テキストを含める (デフォルト: 除外)
