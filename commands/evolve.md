---
name: evolve
description: 関連するインスティンクトをスキル、コマンド、またはエージェントにクラスタリングします
command: true
---

# Evolveコマンド

## 実装

プラグインルートパスを使用してインスティンクトCLIを実行:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" evolve [--generate]
```

または `CLAUDE_PLUGIN_ROOT` が設定されていない場合 (手動インストール):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py evolve [--generate]
```

インスティンクトを分析し、関連するものをより高いレベルの構造にクラスタリング:
- **コマンド**: ユーザーが呼び出すアクションを記述するインスティンクト
- **スキル**: 自動トリガーされた動作を記述するインスティンクト
- **エージェント**: 複雑で複数ステップのプロセスを記述するインスティンクト

## 使用法

```
/evolve                    # すべてのインスティンクトを分析して進化を提案
/evolve --domain testing   # テストドメイン内のインスティンクトのみ進化
/evolve --dry-run          # 作成せずに作成されるものを表示
/evolve --threshold 5      # クラスタリングに5つ以上の関連インスティンクトが必要
```

## 進化ルール

### → コマンド (ユーザー呼び出し)
ユーザーが明示的にリクエストするアクションを記述するインスティンクト:
- 「ユーザーが...するよう要求したとき」についての複数のインスティンクト
- 「新しいXを作成するとき」のようなトリガーを持つインスティンクト
- 繰り返し可能なシーケンスに従うインスティンクト

例:
- `new-table-step1`: 「データベーステーブルを追加するとき、マイグレーションを作成」
- `new-table-step2`: 「データベーステーブルを追加するとき、スキーマを更新」
- `new-table-step3`: 「データベーステーブルを追加するとき、タイプを再生成」

→ 作成: `/new-table` コマンド

### → スキル (自動トリガー)
自動的に発生すべき動作を記述するインスティンクト:
- パターンマッチングトリガー
- エラーハンドリング応答
- コードスタイル強制

例:
- `prefer-functional`: 「関数を書くとき、関数型スタイルを優先」
- `use-immutable`: 「状態を変更するとき、イミュータブルパターンを使用」
- `avoid-classes`: 「モジュールを設計するとき、クラスベースの設計を避ける」

→ 作成: `functional-patterns` スキル

### → エージェント (深さ/隔離が必要)
隔離から恩恵を受ける複雑で複数ステップのプロセスを記述するインスティンクト:
- デバッグワークフロー
- リファクタリング手順
- リサーチタスク

例:
- `debug-step1`: 「デバッグするとき、まずログをチェック」
- `debug-step2`: 「デバッグするとき、失敗するコンポーネントを隔離」
- `debug-step3`: 「デバッグするとき、最小限の再現を作成」
- `debug-step4`: 「デバッグするとき、テストで修正を検証」

→ 作成: `debugger` エージェント

## やること

1. `~/.claude/homunculus/instincts/` からすべてのインスティンクトを読み込む
2. インスティンクトを以下でグループ化:
   - ドメインの類似性
   - トリガーパターンの重複
   - アクションシーケンスの関係
3. 3つ以上の関連インスティンクトの各クラスターについて:
   - 進化のタイプを決定 (コマンド/スキル/エージェント)
   - 適切なファイルを生成
   - `~/.claude/homunculus/evolved/{commands,skills,agents}/` に保存
4. 進化した構造をソースインスティンクトにリンク

## 出力形式

```
🧬 進化分析
==================

進化の準備ができた 3 つのクラスターが見つかりました:

## クラスター 1: データベースマイグレーションワークフロー
インスティンクト: new-table-migration, update-schema, regenerate-types
タイプ: コマンド
信頼度: 85% (12 個の観察に基づく)

作成される: /new-table コマンド
ファイル:
  - ~/.claude/homunculus/evolved/commands/new-table.md

## クラスター 2: 関数型コードスタイル
インスティンクト: prefer-functional, use-immutable, avoid-classes, pure-functions
タイプ: スキル
信頼度: 78% (8 個の観察に基づく)

作成される: functional-patterns スキル
ファイル:
  - ~/.claude/homunculus/evolved/skills/functional-patterns.md

## クラスター 3: デバッグプロセス
インスティンクト: debug-check-logs, debug-isolate, debug-reproduce, debug-verify
タイプ: エージェント
信頼度: 72% (6 個の観察に基づく)

作成される: debugger エージェント
ファイル:
  - ~/.claude/homunculus/evolved/agents/debugger.md

---
これらのファイルを作成するには `/evolve --execute` を実行してください。
```

## フラグ

- `--execute`: 実際に進化した構造を作成 (デフォルトはプレビュー)
- `--dry-run`: 作成せずにプレビュー
- `--domain <name>`: 指定されたドメイン内のインスティンクトのみ進化
- `--threshold <n>`: クラスターを形成するために必要な最小インスティンクト (デフォルト: 3)
- `--type <command|skill|agent>`: 指定されたタイプのみ作成

## 生成されたファイル形式

### コマンド
```markdown
---
name: new-table
description: マイグレーション、スキーマ更新、タイプ生成を含む新しいデータベーステーブルを作成
command: /new-table
evolved_from:
  - new-table-migration
  - update-schema
  - regenerate-types
---

# 新しいテーブルコマンド

[クラスタリングされたインスティンクトに基づいて生成されたコンテンツ]

## ステップ
1. ...
2. ...
```

### スキル
```markdown
---
name: functional-patterns
description: 関数型プログラミングパターンを強制
evolved_from:
  - prefer-functional
  - use-immutable
  - avoid-classes
---

# 関数型パターンスキル

[クラスタリングされたインスティンクトに基づいて生成されたコンテンツ]
```

### エージェント
```markdown
---
name: debugger
description: 体系的なデバッグエージェント
model: sonnet
evolved_from:
  - debug-check-logs
  - debug-isolate
  - debug-reproduce
---

# デバッガーエージェント

[クラスタリングされたインスティンクトに基づいて生成されたコンテンツ]
```
