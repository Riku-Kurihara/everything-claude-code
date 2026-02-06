---
description: あなたの好みのパッケージマネージャーを設定してください (npm/pnpm/yarn/bun)
disable-model-invocation: true
---

# パッケージマネージャー設定

このプロジェクトまたはグローバルに、あなたの好みのパッケージマネージャーを設定してください。

## 使用法

```bash
# 現在のパッケージマネージャーを検出
node scripts/setup-package-manager.js --detect

# グローバル優先設定を設定
node scripts/setup-package-manager.js --global pnpm

# プロジェクト優先設定を設定
node scripts/setup-package-manager.js --project bun

# 利用可能なパッケージマネージャーをリスト
node scripts/setup-package-manager.js --list
```

## 検出の優先度

使用するパッケージマネージャーを決定するとき、以下の順序でチェックされます:

1. **環境変数**: `CLAUDE_PACKAGE_MANAGER`
2. **プロジェクト設定**: `.claude/package-manager.json`
3. **package.json**: `packageManager` フィールド
4. **ロックファイル**: package-lock.json、yarn.lock、pnpm-lock.yaml、bun.lockb の存在
5. **グローバル設定**: `~/.claude/package-manager.json`
6. **フォールバック**: 最初に利用可能なパッケージマネージャー (pnpm > bun > yarn > npm)

## Configuration Files

### Global Configuration
```json
// ~/.claude/package-manager.json
{
  "packageManager": "pnpm"
}
```

### Project Configuration
```json
// .claude/package-manager.json
{
  "packageManager": "bun"
}
```

### package.json
```json
{
  "packageManager": "pnpm@8.6.0"
}
```

## 環境変数

`CLAUDE_PACKAGE_MANAGER` を設定して他のすべての検出方法をオーバーライド:

```bash
# Windows (PowerShell)
$env:CLAUDE_PACKAGE_MANAGER = "pnpm"

# macOS/Linux
export CLAUDE_PACKAGE_MANAGER=pnpm
```

## 検出を実行

現在のパッケージマネージャー検出結果を確認するには、以下を実行:

```bash
node scripts/setup-package-manager.js --detect
```
