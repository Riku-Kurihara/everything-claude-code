---
name: skill-create
description: ローカルの git 履歴を分析してコーディングパターンを抽出し、SKILL.md ファイルを生成します。スキルクリエーター GitHub アプリのローカル版。
allowed_tools: ["Bash", "Read", "Write", "Grep", "Glob"]
---

# /skill-create - ローカルスキル生成

リポジトリの git 履歴を分析してコーディングパターンを抽出し、Claude にあなたのチームの実践を教える SKILL.md ファイルを生成します。

## 使用法

```bash
/skill-create                    # 現在のリポを分析
/skill-create --commits 100      # 最後の 100 個のコミットを分析
/skill-create --output ./skills  # カスタム出力ディレクトリ
/skill-create --instincts        # また continuous-learning-v2 用のインスティンクトを生成
```

## 何をするか

1. **Git 履歴を解析** - コミット、ファイル変更、パターンを分析
2. **パターンを検出** - 繰り返されるワークフローと規約を特定
3. **SKILL.md を生成** - 有効な Claude Code スキルファイルを作成
4. **オプションでインスティンクトを作成** - continuous-learning-v2 システム用

## 分析ステップ

### ステップ 1: Git データを収集

```bash
# Get recent commits with file changes
git log --oneline -n ${COMMITS:-200} --name-only --pretty=format:"%H|%s|%ad" --date=short

# Get commit frequency by file
git log --oneline -n 200 --name-only | grep -v "^$" | grep -v "^[a-f0-9]" | sort | uniq -c | sort -rn | head -20

# Get commit message patterns
git log --oneline -n 200 | cut -d' ' -f2- | head -50
```

### ステップ 2: パターンを検出

これらのパターンタイプを探します:

| Pattern | Detection Method |
|---------|-----------------|
| **Commit conventions** | Regex on commit messages (feat:, fix:, chore:) |
| **File co-changes** | Files that always change together |
| **Workflow sequences** | Repeated file change patterns |
| **Architecture** | Folder structure and naming conventions |
| **Testing patterns** | Test file locations, naming, coverage |

### ステップ 3: SKILL.md を生成

出力形式:

```markdown
---
name: {repo-name}-patterns
description: Coding patterns extracted from {repo-name}
version: 1.0.0
source: local-git-analysis
analyzed_commits: {count}
---

# {Repo Name} Patterns

## Commit Conventions
{detected commit message patterns}

## Code Architecture
{detected folder structure and organization}

## Workflows
{detected repeating file change patterns}

## Testing Patterns
{detected test conventions}
```

### ステップ 4: インスティンクトを生成 (--instincts の場合)

continuous-learning-v2 統合の場合:

```yaml
---
id: {repo}-commit-convention
trigger: "when writing a commit message"
confidence: 0.8
domain: git
source: local-repo-analysis
---

# Use Conventional Commits

## Action
Prefix commits with: feat:, fix:, chore:, docs:, test:, refactor:

## Evidence
- Analyzed {n} commits
- {percentage}% follow conventional commit format
```

## 出力例

TypeScript プロジェクトで `/skill-create` を実行すると以下が生成されることがあります:

```markdown
---
name: my-app-patterns
description: Coding patterns from my-app repository
version: 1.0.0
source: local-git-analysis
analyzed_commits: 150
---

# My App Patterns

## Commit Conventions

This project uses **conventional commits**:
- `feat:` - New features
- `fix:` - Bug fixes
- `chore:` - Maintenance tasks
- `docs:` - Documentation updates

## Code Architecture

```
src/
├── components/     # React components (PascalCase.tsx)
├── hooks/          # Custom hooks (use*.ts)
├── utils/          # Utility functions
├── types/          # TypeScript type definitions
└── services/       # API and external services
```

## Workflows

### Adding a New Component
1. Create `src/components/ComponentName.tsx`
2. Add tests in `src/components/__tests__/ComponentName.test.tsx`
3. Export from `src/components/index.ts`

### Database Migration
1. Modify `src/db/schema.ts`
2. Run `pnpm db:generate`
3. Run `pnpm db:migrate`

## Testing Patterns

- Test files: `__tests__/` directories or `.test.ts` suffix
- Coverage target: 80%+
- Framework: Vitest
```

## GitHub アプリ統合

高度な機能 (10k+ コミット、チーム共有、自動 PR) については、[Skill Creator GitHub アプリ](https://github.com/apps/skill-creator)を使用してください:

- インストール: [github.com/apps/skill-creator](https://github.com/apps/skill-creator)
- 任意の issue に `/skill-creator analyze` とコメント
- 生成されたスキル付きの PR を受け取ります

## 関連コマンド

- `/instinct-import` - 生成されたインスティンクトをインポート
- `/instinct-status` - 学習したインスティンクトを表示
- `/evolve` - インスティンクトをスキル/エージェントにクラスタリング

---

*[Everything Claude Code](https://github.com/affaan-m/everything-claude-code) の一部*
