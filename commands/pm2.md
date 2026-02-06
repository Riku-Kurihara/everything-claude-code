# PM2 初期化

プロジェクトを自動分析して PM2 サービスコマンドを生成します。

**コマンド**: `$ARGUMENTS`

---

## ワークフロー

1. PM2 をチェック (不足している場合は `npm install -g pm2` でインストール)
2. プロジェクトをスキャンしてサービスを特定 (フロントエンド/バックエンド/データベース)
3. 設定ファイルと個別のコマンドファイルを生成

---

## サービス検出

| タイプ | 検出 | デフォルトポート |
|------|-----------|--------------|
| Vite | vite.config.* | 5173 |
| Next.js | next.config.* | 3000 |
| Nuxt | nuxt.config.* | 3000 |
| CRA | react-scripts in package.json | 3000 |
| Express/Node | server/backend/api ディレクトリ + package.json | 3000 |
| FastAPI/Flask | requirements.txt / pyproject.toml | 8000 |
| Go | go.mod / main.go | 8080 |

**ポート検出の優先度**: ユーザー指定 > .env > 設定ファイル > スクリプト引数 > デフォルトポート

---

## Generated Files

```
project/
├── ecosystem.config.cjs              # PM2 config
├── {backend}/start.cjs               # Python wrapper (if applicable)
└── .claude/
    ├── commands/
    │   ├── pm2-all.md                # Start all + monit
    │   ├── pm2-all-stop.md           # Stop all
    │   ├── pm2-all-restart.md        # Restart all
    │   ├── pm2-{port}.md             # Start single + logs
    │   ├── pm2-{port}-stop.md        # Stop single
    │   ├── pm2-{port}-restart.md     # Restart single
    │   ├── pm2-logs.md               # View all logs
    │   └── pm2-status.md             # View status
    └── scripts/
        ├── pm2-logs-{port}.ps1       # Single service logs
        └── pm2-monit.ps1             # PM2 monitor
```

---

## Windows Configuration (IMPORTANT)

### ecosystem.config.cjs

**Must use `.cjs` extension**

```javascript
module.exports = {
  apps: [
    // Node.js (Vite/Next/Nuxt)
    {
      name: 'project-3000',
      cwd: './packages/web',
      script: 'node_modules/vite/bin/vite.js',
      args: '--port 3000',
      interpreter: 'C:/Program Files/nodejs/node.exe',
      env: { NODE_ENV: 'development' }
    },
    // Python
    {
      name: 'project-8000',
      cwd: './backend',
      script: 'start.cjs',
      interpreter: 'C:/Program Files/nodejs/node.exe',
      env: { PYTHONUNBUFFERED: '1' }
    }
  ]
}
```

**Framework script paths:**

| Framework | script | args |
|-----------|--------|------|
| Vite | `node_modules/vite/bin/vite.js` | `--port {port}` |
| Next.js | `node_modules/next/dist/bin/next` | `dev -p {port}` |
| Nuxt | `node_modules/nuxt/bin/nuxt.mjs` | `dev --port {port}` |
| Express | `src/index.js` or `server.js` | - |

### Python Wrapper Script (start.cjs)

```javascript
const { spawn } = require('child_process');
const proc = spawn('python', ['-m', 'uvicorn', 'app.main:app', '--host', '0.0.0.0', '--port', '8000', '--reload'], {
  cwd: __dirname, stdio: 'inherit', windowsHide: true
});
proc.on('close', (code) => process.exit(code));
```

---

## Command File Templates (Minimal Content)

### pm2-all.md (Start all + monit)
```markdown
Start all services and open PM2 monitor.
\`\`\`bash
cd "{PROJECT_ROOT}" && pm2 start ecosystem.config.cjs && start wt.exe -d "{PROJECT_ROOT}" pwsh -NoExit -c "pm2 monit"
\`\`\`
```

### pm2-all-stop.md
```markdown
Stop all services.
\`\`\`bash
cd "{PROJECT_ROOT}" && pm2 stop all
\`\`\`
```

### pm2-all-restart.md
```markdown
Restart all services.
\`\`\`bash
cd "{PROJECT_ROOT}" && pm2 restart all
\`\`\`
```

### pm2-{port}.md (Start single + logs)
```markdown
Start {name} ({port}) and open logs.
\`\`\`bash
cd "{PROJECT_ROOT}" && pm2 start ecosystem.config.cjs --only {name} && start wt.exe -d "{PROJECT_ROOT}" pwsh -NoExit -c "pm2 logs {name}"
\`\`\`
```

### pm2-{port}-stop.md
```markdown
Stop {name} ({port}).
\`\`\`bash
cd "{PROJECT_ROOT}" && pm2 stop {name}
\`\`\`
```

### pm2-{port}-restart.md
```markdown
Restart {name} ({port}).
\`\`\`bash
cd "{PROJECT_ROOT}" && pm2 restart {name}
\`\`\`
```

### pm2-logs.md
```markdown
View all PM2 logs.
\`\`\`bash
cd "{PROJECT_ROOT}" && pm2 logs
\`\`\`
```

### pm2-status.md
```markdown
View PM2 status.
\`\`\`bash
cd "{PROJECT_ROOT}" && pm2 status
\`\`\`
```

### PowerShell Scripts (pm2-logs-{port}.ps1)
```powershell
Set-Location "{PROJECT_ROOT}"
pm2 logs {name}
```

### PowerShell Scripts (pm2-monit.ps1)
```powershell
Set-Location "{PROJECT_ROOT}"
pm2 monit
```

---

## Key Rules

1. **Config file**: `ecosystem.config.cjs` (not .js)
2. **Node.js**: Specify bin path directly + interpreter
3. **Python**: Node.js wrapper script + `windowsHide: true`
4. **Open new window**: `start wt.exe -d "{path}" pwsh -NoExit -c "command"`
5. **Minimal content**: Each command file has only 1-2 lines description + bash block
6. **Direct execution**: No AI parsing needed, just run the bash command

---

## 実行

`$ARGUMENTS` に基づいて、初期化を実行:

1. プロジェクトをスキャンしてサービスを検出
2. `ecosystem.config.cjs` を生成
3. Python サービス用に `{backend}/start.cjs` を生成 (該当する場合)
4. `.claude/commands/` にコマンドファイルを生成
5. `.claude/scripts/` にスクリプトファイルを生成
6. **プロジェクト CLAUDE.md を更新** PM2 情報で (下記を参照)
7. **完了サマリーを表示** ターミナルコマンド付き

---

## 初期化後: CLAUDE.md を更新

ファイル生成後、プロジェクトの `CLAUDE.md` に PM2 セクションを追加 (存在しない場合は作成):

```markdown
## PM2 Services

| Port | Name | Type |
|------|------|------|
| {port} | {name} | {type} |

**Terminal Commands:**
```bash
pm2 start ecosystem.config.cjs   # First time
pm2 start all                    # After first time
pm2 stop all / pm2 restart all
pm2 start {name} / pm2 stop {name}
pm2 logs / pm2 status / pm2 monit
pm2 save                         # Save process list
pm2 resurrect                    # Restore saved list
```
```

**CLAUDE.md 更新ルール:**
- PM2 セクションが存在する場合は置き換え
- 存在しない場合は末尾に追加
- コンテンツは最小限で本質的に保つ

---

## 初期化後: サマリーを表示

すべてのファイルが生成された後、出力:

```
## PM2 Init Complete

**Services:**
| Port | Name | Type |
|------|------|------|
| {port} | {name} | {type} |

**Claude Commands:** /pm2-all, /pm2-all-stop, /pm2-{port}, /pm2-{port}-stop, /pm2-logs, /pm2-status

**Terminal Commands:**
# First time (with config file)
pm2 start ecosystem.config.cjs && pm2 save

# After first time (simplified)
pm2 start all          # Start all
pm2 stop all           # Stop all
pm2 restart all        # Restart all
pm2 start {name}       # Start single
pm2 stop {name}        # Stop single
pm2 logs               # View logs
pm2 monit              # Monitor panel
pm2 resurrect          # Restore saved processes

**Tip:** Run `pm2 save` after first start to enable simplified commands.
```
