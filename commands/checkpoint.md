# チェックポイントコマンド

ワークフロー内にチェックポイントを作成または検証します。

## 使用法

`/checkpoint [create|verify|list] [name]`

## チェックポイントの作成

チェックポイントを作成するとき:

1. `/verify quick` を実行して現在の状態がクリーンであることを確認
2. チェックポイント名でgitスタッシュまたはコミットを作成
3. `.claude/checkpoints.log` にチェックポイントを記録:

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD)" >> .claude/checkpoints.log
```

4. チェックポイント作成を報告

## チェックポイントの検証

チェックポイントに対して検証するとき:

1. ログからチェックポイントを読み込む
2. 現在の状態をチェックポイントと比較:
   - チェックポイント以降に追加されたファイル
   - チェックポイント以降に変更されたファイル
   - テスト成功率 (現在と過去)
   - カバレッジ (現在と過去)

3. 報告:
```
チェックポイント比較: $NAME
============================
ファイル変更数: X
テスト: +Y 成功 / -Z 失敗
カバレッジ: +X% / -Y%
ビルド: [成功/失敗]
```

## チェックポイント一覧

すべてのチェックポイントを表示:
- 名前
- タイムスタンプ
- Git SHA
- ステータス (現在, 背後, 前方)

## ワークフロー

典型的なチェックポイントフロー:

```
[開始] --> /checkpoint create "feature-start"
   |
[実装] --> /checkpoint create "core-done"
   |
[テスト] --> /checkpoint verify "core-done"
   |
[リファクタ] --> /checkpoint create "refactor-done"
   |
[PR] --> /checkpoint verify "feature-start"
```

## 引数

$ARGUMENTS:
- `create <name>` - 名前付きチェックポイントを作成
- `verify <name>` - 名前付きチェックポイントに対して検証
- `list` - すべてのチェックポイントを表示
- `clear` - 古いチェックポイントを削除 (最後の5つを保持)
