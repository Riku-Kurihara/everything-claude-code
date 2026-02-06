# ルール

## 構成

ルールは**共通**レイヤーと**言語別**ディレクトリで構成されています：

```
rules/
├── common/          # 言語に依存しない原則 (常にインストール)
│   ├── coding-style.md
│   ├── git-workflow.md
│   ├── testing.md
│   ├── performance.md
│   ├── patterns.md
│   ├── hooks.md
│   ├── agents.md
│   └── security.md
├── typescript/      # TypeScript/JavaScript特有
├── python/          # Python特有
└── golang/          # Go特有
```

- **common/** には言語に依存しない汎用の原則を含みます — 言語特有のコード例はありません。
- **言語ディレクトリ** は共通ルールを拡張し、フレームワーク固有のパターン、ツール、コード例を追加します。各ファイルは対応する共通ファイルを参照します。

## インストール

```bash
# 共通ルールをインストール (すべてのプロジェクトで必須)
cp -r rules/common/* ~/.claude/rules/

# 言語別ルールをプロジェクトのテックスタックに基づいてインストール
cp -r rules/typescript/* ~/.claude/rules/
cp -r rules/python/* ~/.claude/rules/
cp -r rules/golang/* ~/.claude/rules/

# 注意！！！ 実際のプロジェクト要件に応じて設定してください。ここでの設定は参考用です。

```

## ルール vs スキル

- **ルール** は広く適用される標準、慣例、チェックリストを定義します（例：「テストカバレッジ 80%」、「ハードコードされたシークレットなし」）。
- **スキル** (`skills/` ディレクトリ) は特定のタスクの深い実用的なリファレンス資料を提供します（例：`python-patterns`、`golang-testing`）。

言語別ルールファイルは必要に応じて関連するスキルを参照します。ルールは**何を**するかを示し、スキルは**どのように**するかを示します。

## 新しい言語の追加

新しい言語のサポートを追加するには（例：`rust/`）：

1. `rules/rust/` ディレクトリを作成
2. 共通ルールを拡張するファイルを追加：
   - `coding-style.md` — フォーマッティングツール、イディオム、エラーハンドリングパターン
   - `testing.md` — テストフレームワーク、カバレッジツール、テスト構成
   - `patterns.md` — 言語特有のデザインパターン
   - `hooks.md` — フォーマッター、リンター、型チェッカー用の PostToolUse フック
   - `security.md` — シークレット管理、セキュリティスキャンツール
3. 各ファイルは以下で始まります：
   ```
   > This file extends [common/xxx.md](../common/xxx.md) with <Language> specific content.
   ```
4. 利用可能な場合は既存のスキルを参照するか、`skills/` の下に新しいものを作成します。
