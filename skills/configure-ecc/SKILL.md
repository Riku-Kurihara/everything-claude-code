---
name: configure-ecc
description: Everything Claude Codeの対話型インストーラー — ユーザーレベルまたはプロジェクトレベルのディレクトリへのスキルとルールの選択的インストールをガイドし、パスを検証し、オプションでインストール済みファイルを最適化します。
---

# Everything Claude Code(ECC)の設定

Everything Claude Codeプロジェクトの対話型ステップバイステップインストレーションウィザード。AskUserQuestionを使用してスキルとルールの選択的インストールをガイドし、正確性を検証して最適化を提供します。

## アクティベーション時期

- ユーザーが「configure ecc」、「install ecc」、「setup everything claude code」など同様の表現をする
- ユーザーがこのプロジェクトからスキルまたはルールを選択的にインストールしたい場合
- ユーザーが既存のECCインストールを検証または修正したい場合
- ユーザーがインストール済みのスキルまたはルールをプロジェクト用に最適化したい場合

## 前提条件

このスキルはアクティベーション前にClaude Codeでアクセス可能である必要があります。ブートストラップ方法は2つあります:
1. **プラグイン経由**: `/plugin install everything-claude-code` — プラグインがこのスキルを自動的にロード
2. **手動**: `~/.claude/skills/configure-ecc/SKILL.md`にこのスキルのみをコピーしてから、「configure ecc」と言ってアクティベート

---

## ステップ0: ECCリポジトリをクローン

インストール前に、最新のECCソースを`/tmp`にクローン:

```bash
rm -rf /tmp/everything-claude-code
git clone https://github.com/affaan-m/everything-claude-code.git /tmp/everything-claude-code
```

`ECC_ROOT=/tmp/everything-claude-code`を後続のすべてのコピー操作のソースとして設定します。

クローンが失敗した場合(ネットワーク問題など)、`AskUserQuestion`を使用してユーザーに既存のECCクローンへのローカルパスを提供するよう依頼します。

---

## ステップ1: インストールレベルを選択

`AskUserQuestion`を使用してユーザーにインストール場所を尋ねます:

```
質問: "ECCコンポーネントはどこにインストールしますか?"
オプション:
  - "ユーザーレベル(~/.claude/)" — "すべてのClaude Codeプロジェクトに適用"
  - "プロジェクトレベル(.claude/)" — "現在のプロジェクトのみに適用"
  - "両方" — "共通/共有項目はユーザーレベル、プロジェクト固有項目はプロジェクトレベル"
```

選択を`INSTALL_LEVEL`として保存します。ターゲットディレクトリを設定:
- ユーザーレベル: `TARGET=~/.claude`
- プロジェクトレベル: `TARGET=.claude` (現在のプロジェクトルートからの相対)
- 両方: `TARGET_USER=~/.claude`, `TARGET_PROJECT=.claude`

必要に応じてターゲットディレクトリを作成:
```bash
mkdir -p $TARGET/skills $TARGET/rules
```

---

## ステップ2: スキルを選択してインストール

### 2a: スキルカテゴリを選択

27個のスキルが4つのカテゴリに編成されています。`multiSelect: true`で`AskUserQuestion`を使用:

```
質問: "どのスキルカテゴリをインストールしたいですか?"
オプション:
  - "フレームワークと言語" — "Django、Spring Boot、Go、Python、Java、フロントエンド、バックエンドパターン"
  - "データベース" — "PostgreSQL、ClickHouse、JPA/Hibernateパターン"
  - "ワークフローと品質" — "TDD、検証、学習、セキュリティレビュー、圧縮"
  - "すべてのスキル" — "利用可能なすべてのスキルをインストール"
```

### 2b: 個別スキルを確認

選択した各カテゴリについて、以下のスキルの完全なリストを印刷して、ユーザーに具体的なスキルを確認またはデセレクトするよう尋ねます。リストが4項目を超える場合、リストをテキストとして印刷し、`AskUserQuestion`を「すべてリストをインストール」オプション付きで使用し、ユーザーが特定の名前を貼り付けるための「その他」オプションを含めます。

**カテゴリ: フレームワークと言語(16個のスキル)**

| スキル | 説明 |
|-------|------|
| `backend-patterns` | バックエンド設計、API設計、Node.js/Express/Next.jsのサーバーサイドベストプラクティス |
| `coding-standards` | TypeScript、JavaScript、React、Node.js開発向けの普遍的なコーディング基準 |
| `django-patterns` | Django設計、DRF REST API、ORM、キャッシング、シグナル、ミドルウェア |
| `django-security` | Djangoセキュリティ: 認証、CSRF、SQLインジェクション、XSS防止 |
| `django-tdd` | Django テスト: pytest-django、factory_boy、モッキング、カバレッジ |
| `django-verification` | Django検証ループ: マイグレーション、リント、テスト、セキュリティスキャン |
| `frontend-patterns` | React、Next.js、状態管理、パフォーマンス、UIパターン |
| `golang-patterns` | 慣用的なGoパターン、堅牢なGoアプリケーション向け慣例 |
| `golang-testing` | Go テスト: テーブル駆動テスト、サブテスト、ベンチマーク、ファジング |
| `java-coding-standards` | Spring Boot向けJavaコーディング基準: 命名、イミュータビリティ、Optional、ストリーム |
| `python-patterns` | Python慣用的な使い方、PEP 8、型ヒント、ベストプラクティス |
| `python-testing` | Python テスト: pytest、TDD、フィクスチャ、モッキング、パラメータ化 |
| `springboot-patterns` | Spring Bootの設計、REST API、レイヤー化サービス、キャッシング、非同期 |
| `springboot-security` | Spring Security: 認証/認可、検証、CSRF、シークレット、レート制限 |
| `springboot-tdd` | Spring Boot TDD: JUnit 5、Mockito、MockMvc、Testcontainers |
| `springboot-verification` | Spring Boot検証: ビルド、静的分析、テスト、セキュリティスキャン |

**カテゴリ: データベース(3個のスキル)**

| スキル | 説明 |
|-------|------|
| `clickhouse-io` | ClickHouseパターン、クエリ最適化、分析、データエンジニアリング |
| `jpa-patterns` | JPA/Hibernateエンティティ設計、リレーションシップ、クエリ最適化、トランザクション |
| `postgres-patterns` | PostgreSQLクエリ最適化、スキーマ設計、インデックス、セキュリティ |

**カテゴリ: ワークフローと品質(8個のスキル)**

| スキル | 説明 |
|-------|------|
| `continuous-learning` | セッションから再利用可能なパターンを自動抽出して学習スキルとして保存 |
| `continuous-learning-v2` | 信頼度スコアリング付き本能ベース学習、スキル/コマンド/エージェントに進化 |
| `eval-harness` | 評価駆動開発(EDD)向け正式評価フレームワーク |
| `iterative-retrieval` | サブエージェントコンテキスト問題向けプログレッシブコンテキスト精製 |
| `security-review` | セキュリティチェックリスト: 認証、入力、シークレット、API、決済機能 |
| `strategic-compact` | 論理的な間隔での手動コンテキスト圧縮を提案 |
| `tdd-workflow` | TDDを強制: 80%以上カバレッジ: ユニット、統合、E2E |
| `verification-loop` | 検証とクオリティ品質ループパターン |

**スタンドアロン**

| スキル | 説明 |
|-------|------|
| `project-guidelines-example` | プロジェクト固有スキル作成向けテンプレート |

### 2c: インストールを実行

選択した各スキルについて、スキル全体ディレクトリをコピー:
```bash
cp -r $ECC_ROOT/skills/<skill-name> $TARGET/skills/
```

注: `continuous-learning`と`continuous-learning-v2`には追加ファイル(config.json、フック、スクリプト)があります — SKILL.mdだけでなく、ディレクトリ全体をコピーしてください。

---

## ステップ3: ルールを選択してインストール

`multiSelect: true`で`AskUserQuestion`を使用:

```
質問: "どのルールセットをインストールしたいですか?"
オプション:
  - "共通ルール(推奨)" — "言語非依存の原則: コーディング様式、Gitワークフロー、テスト、セキュリティなど(8ファイル)"
  - "TypeScript/JavaScript" — "TS/JSパターン、フック、Playwright付きテスト(5ファイル)"
  - "Python" — "Pythonパターン、pytest、black/ruff書式(5ファイル)"
  - "Go" — "Goパターン、テーブル駆動テスト、gofmt/staticcheck(5ファイル)"
```

インストール実行:
```bash
# 共通ルール(rules/にフラットコピー)
cp -r $ECC_ROOT/rules/common/* $TARGET/rules/

# 言語固有ルール(rules/にフラットコピー)
cp -r $ECC_ROOT/rules/typescript/* $TARGET/rules/   # 選択時
cp -r $ECC_ROOT/rules/python/* $TARGET/rules/        # 選択時
cp -r $ECC_ROOT/rules/golang/* $TARGET/rules/        # 選択時
```

**重要**: ユーザーが共通ルールなしで言語固有ルールを選択した場合、警告:
> "言語固有ルールは共通ルールを拡張します。共通ルールなしでインストールすると、カバレッジが不完全になる可能性があります。共通ルールもインストールしますか?"

---

## ステップ4: インストール後の検証

インストール後、これらの自動チェックを実行:

### 4a: ファイル存在を検証

インストール済みファイルのリストを表示し、ターゲット場所に存在することを確認:
```bash
ls -la $TARGET/skills/
ls -la $TARGET/rules/
```

### 4b: パス参照を確認

インストール済みのすべての`.md`ファイルをスキャンしてパス参照:
```bash
grep -rn "~/.claude/" $TARGET/skills/ $TARGET/rules/
grep -rn "../common/" $TARGET/rules/
grep -rn "skills/" $TARGET/skills/
```

**プロジェクトレベルインストールの場合**、`~/.claude/`パスへの参照にフラグを立てます:
- スキルが`~/.claude/settings.json`を参照している場合 — これは通常問題なし(設定は常にユーザーレベル)
- スキルが`~/.claude/skills/`または`~/.claude/rules/`を参照している場合 — プロジェクトレベルのみでインストールされている場合、これは破損する可能性があります
- スキルが別のスキルを名前で参照している場合 — 参照されたスキルもインストールされたことを確認

### 4c: スキル間のクロスリファレンスを確認

一部のスキルは他のスキルを参照しています。これらの依存関係を検証:
- `django-tdd`は`django-patterns`を参照できます
- `springboot-tdd`は`springboot-patterns`を参照できます
- `continuous-learning-v2`は`~/.claude/homunculus/`ディレクトリを参照
- `python-testing`は`python-patterns`を参照できます
- `golang-testing`は`golang-patterns`を参照できます
- 言語固有ルールは`common/`カウンターパートを参照

### 4d: 問題をレポート

見つかった問題ごとに、レポート:
1. **ファイル**: 問題のある参照を含むファイル
2. **行**: 行番号
3. **問題**: 何が間違っているか(例: "python-patternsをインストールしていないがpython-patternsスキルを参照している")
4. **推奨修正**: やるべきこと(例: "python-patternsスキルをインストール"または"パスを.claude/skills/に更新")

---

## ステップ5: インストール済みファイルの最適化(オプション)

`AskUserQuestion`を使用:

```
質問: "インストール済みファイルをプロジェクト用に最適化しますか?"
オプション:
  - "スキルを最適化" — "関連性のないセクション削除、パス調整、テックスタック合わせ"
  - "ルールを最適化" — "カバレッジ目標調整、プロジェクト固有パターン追加、ツール設定カスタマイズ"
  - "両方を最適化" — "インストール済みファイルの完全最適化"
  - "スキップ" — "すべてをそのままに"
```

### スキル最適化時:
1. インストール済み各SKILL.mdを読む
2. プロジェクトのテックスタックが何かをユーザーに尋ねる(まだ知らない場合)
3. 各スキルについて、関連性のないセクションの削除を提案
4. SKILL.mdファイルをインストールターゲット場所で即座に編集(ソースリポではなく)
5. ステップ4で見つかったパス問題を修正

### ルール最適化時:
1. インストール済み各ルール.mdファイルを読む
2. ユーザーに環境設定を尋ねます:
   - テストカバレッジ目標(デフォルト80%)
   - 推奨書式ツール
   - Gitワークフロー慣例
   - セキュリティ要件
3. インストールターゲットでルールファイルを即座に編集

**重要**: インストールターゲット(`$TARGET/`)のファイルのみ変更、ソースECCリポジトリ(`$ECC_ROOT/`)のファイルは決して変更しないください。

---

## ステップ6: インストール概要

`/tmp`からクローンリポジトリをクリーンアップ:

```bash
rm -rf /tmp/everything-claude-code
```

概要レポートを印刷:

```
## ECCインストール完了

### インストールターゲット
- レベル: [ユーザーレベル / プロジェクトレベル / 両方]
- パス: [ターゲットパス]

### インストール済みスキル([数])
- skill-1, skill-2, skill-3, ...

### インストール済みルール([数])
- common (8ファイル)
- typescript (5ファイル)
- ...

### 検証結果
- [数]個の問題が見つかり、[数]個が修正
- [残りの問題のリスト]

### 適用された最適化
- [実施された変更のリスト、または「なし"]
```

---

## トラブルシューティング

### "Claude Codeがスキルをピックアップしていない"
- スキルディレクトリに`SKILL.md`ファイルが含まれていることを確認(ルーズ.mdファイルのみではなく)
- ユーザーレベル: `~/.claude/skills/<skill-name>/SKILL.md`が存在することを確認
- プロジェクトレベル: `.claude/skills/<skill-name>/SKILL.md`が存在することを確認

### "ルールが機能しない"
- ルールはサブディレクトリではなくフラットファイル: `$TARGET/rules/coding-style.md`(正) vs `$TARGET/rules/common/coding-style.md`(フラットインストール向けに不正)
- ルールをインストール後、Claude Codeを再起動

### "プロジェクトレベルインストール後のパス参照エラー"
- 一部スキルは`~/.claude/`パスを想定しています。ステップ4検証を実行して、これらを見つけて修正してください。
- `continuous-learning-v2`の場合、`~/.claude/homunculus/`ディレクトリは常にユーザーレベルです — これは予期されたエラーではありません。
