# Evalコマンド

Eval駆動開発ワークフローを管理します。

## 使用法

`/eval [define|check|report|list] [feature-name]`

## Evalの定義

`/eval define feature-name`

新しいeval定義を作成:

1. テンプレート付きで `.claude/evals/feature-name.md` を作成:

```markdown
## EVAL: feature-name
作成日時: $(date)

### 機能Eval
- [ ] [機能1の説明]
- [ ] [機能2の説明]

### リグレッションEval
- [ ] [既存の動作1がまだ機能しています]
- [ ] [既存の動作2がまだ機能しています]

### 成功基準
- 機能Evalで pass@3 > 90%
- リグレッションEvalで pass^3 = 100%
```

2. ユーザーに具体的な基準を入力するようプロンプト

## Evalの確認

`/eval check feature-name`

機能のevalを実行:

1. `.claude/evals/feature-name.md` からeval定義を読み込む
2. 各機能Evalについて:
   - 基準の確認を試みる
   - PASS/FAILを記録
   - `.claude/evals/feature-name.log` に試行をログ
3. 各リグレッションEvalについて:
   - 関連するテストを実行
   - ベースラインと比較
   - PASS/FAILを記録
4. 現在のステータスを報告:

```
EVAL チェック: feature-name
========================
機能: X/Y 成功
リグレッション: X/Y 成功
ステータス: 進行中 / 準備完了
```

## Evalレポート

`/eval report feature-name`

包括的なevalレポートを生成:

```
EVALレポート: feature-name
=========================
生成日時: $(date)

機能EVAL
--------
[eval-1]: 成功 (pass@1)
[eval-2]: 成功 (pass@2) - 再試行が必要
[eval-3]: 失敗 - 注記を参照

リグレッションEVAL
------------------
[test-1]: 成功
[test-2]: 成功
[test-3]: 成功

メトリクス
---------
機能 pass@1: 67%
機能 pass@3: 100%
リグレッション pass^3: 100%

注記
----
[問題、エッジケース、または観察事項]

推奨事項
-------
[リリース / 作業が必要 / ブロック済み]
```

## Evalの一覧表示

`/eval list`

すべてのeval定義を表示:

```
EVAL定義
================
feature-auth      [3/5 成功] 進行中
feature-search    [5/5 成功] 準備完了
feature-export    [0/4 成功] 未開始
```

## 引数

$ARGUMENTS:
- `define <name>` - 新しいeval定義を作成
- `check <name>` - Evalを実行して確認
- `report <name>` - 完全なレポートを生成
- `list` - すべてのevalを表示
- `clean` - 古いevalログを削除 (最後の10回を保持)
