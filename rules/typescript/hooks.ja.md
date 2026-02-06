# TypeScript/JavaScript フック

> このファイルは [common/hooks.md](../common/hooks.md) を TypeScript/JavaScript 特有の内容で拡張します。

## PostToolUse フック

`~/.claude/settings.json` で設定：

- **Prettier**：編集後の JS/TS ファイルを自動フォーマット
- **TypeScript check**：`.ts`/`.tsx` ファイル編集後に `tsc` を実行
- **console.log 警告**：編集されたファイルの `console.log` について警告

## Stop フック

- **console.log 監査**：セッション終了前に変更されたすべてのファイルの `console.log` をチェック
