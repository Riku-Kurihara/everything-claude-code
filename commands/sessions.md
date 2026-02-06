# セッションコマンド

Claude Code セッション履歴を管理 - `~/.claude/sessions/` に保存されているセッションをリスト、ロード、エイリアス、編集。

## 使用法

`/sessions [list|load|alias|info|help] [options]`

## アクション

### セッション一覧

メタデータ、フィルタリング、ページング付きのすべてのセッションを表示。

```bash
/sessions                              # すべてのセッションをリスト (デフォルト)
/sessions list                         # 上記と同じ
/sessions list --limit 10              # 10 個のセッションを表示
/sessions list --date 2026-02-01       # 日付でフィルタリング
/sessions list --search abc            # セッション ID で検索
```

**Script:**
```bash
node -e "
const sm = require('./scripts/lib/session-manager');
const aa = require('./scripts/lib/session-aliases');

const result = sm.getAllSessions({ limit: 20 });
const aliases = aa.listAliases();
const aliasMap = {};
for (const a of aliases) aliasMap[a.sessionPath] = a.name;

console.log('Sessions (showing ' + result.sessions.length + ' of ' + result.total + '):');
console.log('');
console.log('ID        Date        Time     Size     Lines  Alias');
console.log('────────────────────────────────────────────────────');

for (const s of result.sessions) {
  const alias = aliasMap[s.filename] || '';
  const size = sm.getSessionSize(s.sessionPath);
  const stats = sm.getSessionStats(s.sessionPath);
  const id = s.shortId === 'no-id' ? '(none)' : s.shortId.slice(0, 8);
  const time = s.modifiedTime.toTimeString().slice(0, 5);

  console.log(id.padEnd(8) + ' ' + s.date + '  ' + time + '   ' + size.padEnd(7) + '  ' + String(stats.lineCount).padEnd(5) + '  ' + alias);
}
"
```

### セッションをロード

セッションの内容をロードして表示 (ID またはエイリアスで)。

```bash
/sessions load <id|alias>             # セッションをロード
/sessions load 2026-02-01             # 日付で (ID なしセッションの場合)
/sessions load a1b2c3d4               # 短い ID で
/sessions load my-alias               # エイリアス名で
```

**Script:**
```bash
node -e "
const sm = require('./scripts/lib/session-manager');
const aa = require('./scripts/lib/session-aliases');
const id = process.argv[1];

// First try to resolve as alias
const resolved = aa.resolveAlias(id);
const sessionId = resolved ? resolved.sessionPath : id;

const session = sm.getSessionById(sessionId, true);
if (!session) {
  console.log('Session not found: ' + id);
  process.exit(1);
}

const stats = sm.getSessionStats(session.sessionPath);
const size = sm.getSessionSize(session.sessionPath);
const aliases = aa.getAliasesForSession(session.filename);

console.log('Session: ' + session.filename);
console.log('Path: ~/.claude/sessions/' + session.filename);
console.log('');
console.log('Statistics:');
console.log('  Lines: ' + stats.lineCount);
console.log('  Total items: ' + stats.totalItems);
console.log('  Completed: ' + stats.completedItems);
console.log('  In progress: ' + stats.inProgressItems);
console.log('  Size: ' + size);
console.log('');

if (aliases.length > 0) {
  console.log('Aliases: ' + aliases.map(a => a.name).join(', '));
  console.log('');
}

if (session.metadata.title) {
  console.log('Title: ' + session.metadata.title);
  console.log('');
}

if (session.metadata.started) {
  console.log('Started: ' + session.metadata.started);
}

if (session.metadata.lastUpdated) {
  console.log('Last Updated: ' + session.metadata.lastUpdated);
}
" "$ARGUMENTS"
```

### エイリアスを作成

セッション用に覚えやすいエイリアスを作成。

```bash
/sessions alias <id> <name>           # エイリアスを作成
/sessions alias 2026-02-01 today-work # "today-work" という名前のエイリアスを作成
```

**Script:**
```bash
node -e "
const sm = require('./scripts/lib/session-manager');
const aa = require('./scripts/lib/session-aliases');

const sessionId = process.argv[1];
const aliasName = process.argv[2];

if (!sessionId || !aliasName) {
  console.log('Usage: /sessions alias <id> <name>');
  process.exit(1);
}

// Get session filename
const session = sm.getSessionById(sessionId);
if (!session) {
  console.log('Session not found: ' + sessionId);
  process.exit(1);
}

const result = aa.setAlias(aliasName, session.filename);
if (result.success) {
  console.log('✓ Alias created: ' + aliasName + ' → ' + session.filename);
} else {
  console.log('✗ Error: ' + result.error);
  process.exit(1);
}
" "$ARGUMENTS"
```

### エイリアスを削除

既存のエイリアスを削除。

```bash
/sessions alias --remove <name>        # エイリアスを削除
/sessions unalias <name>               # 上記と同じ
```

**Script:**
```bash
node -e "
const aa = require('./scripts/lib/session-aliases');

const aliasName = process.argv[1];
if (!aliasName) {
  console.log('Usage: /sessions alias --remove <name>');
  process.exit(1);
}

const result = aa.deleteAlias(aliasName);
if (result.success) {
  console.log('✓ Alias removed: ' + aliasName);
} else {
  console.log('✗ Error: ' + result.error);
  process.exit(1);
}
" "$ARGUMENTS"
```

### セッション情報

セッションの詳細情報を表示。

```bash
/sessions info <id|alias>              # セッション詳細を表示
```

**Script:**
```bash
node -e "
const sm = require('./scripts/lib/session-manager');
const aa = require('./scripts/lib/session-aliases');

const id = process.argv[1];
const resolved = aa.resolveAlias(id);
const sessionId = resolved ? resolved.sessionPath : id;

const session = sm.getSessionById(sessionId, true);
if (!session) {
  console.log('Session not found: ' + id);
  process.exit(1);
}

const stats = sm.getSessionStats(session.sessionPath);
const size = sm.getSessionSize(session.sessionPath);
const aliases = aa.getAliasesForSession(session.filename);

console.log('Session Information');
console.log('════════════════════');
console.log('ID:          ' + (session.shortId === 'no-id' ? '(none)' : session.shortId));
console.log('Filename:    ' + session.filename);
console.log('Date:        ' + session.date);
console.log('Modified:    ' + session.modifiedTime.toISOString().slice(0, 19).replace('T', ' '));
console.log('');
console.log('Content:');
console.log('  Lines:         ' + stats.lineCount);
console.log('  Total items:   ' + stats.totalItems);
console.log('  Completed:     ' + stats.completedItems);
console.log('  In progress:   ' + stats.inProgressItems);
console.log('  Size:          ' + size);
if (aliases.length > 0) {
  console.log('Aliases:     ' + aliases.map(a => a.name).join(', '));
}
" "$ARGUMENTS"
```

### エイリアスをリスト

すべてのセッションエイリアスを表示。

```bash
/sessions aliases                      # すべてのエイリアスをリスト
```

**Script:**
```bash
node -e "
const aa = require('./scripts/lib/session-aliases');

const aliases = aa.listAliases();
console.log('Session Aliases (' + aliases.length + '):');
console.log('');

if (aliases.length === 0) {
  console.log('No aliases found.');
} else {
  console.log('Name          Session File                    Title');
  console.log('─────────────────────────────────────────────────────────────');
  for (const a of aliases) {
    const name = a.name.padEnd(12);
    const file = (a.sessionPath.length > 30 ? a.sessionPath.slice(0, 27) + '...' : a.sessionPath).padEnd(30);
    const title = a.title || '';
    console.log(name + ' ' + file + ' ' + title);
  }
}
"
```

## 引数

$ARGUMENTS:
- `list [オプション]` - セッションをリスト
  - `--limit <n>` - 表示するセッションの最大数 (デフォルト: 50)
  - `--date <YYYY-MM-DD>` - 日付でフィルタリング
  - `--search <パターン>` - セッション ID で検索
- `load <id|エイリアス>` - セッション内容をロード
- `alias <id> <名前>` - セッション用エイリアスを作成
- `alias --remove <名前>` - エイリアスを削除
- `unalias <名前>` - `--remove` と同じ
- `info <id|エイリアス>` - セッション統計を表示
- `aliases` - すべてのエイリアスをリスト
- `help` - このヘルプを表示

## 例

```bash
# すべてのセッションをリスト
/sessions list

# 今日のセッション用エイリアスを作成
/sessions alias 2026-02-01 today

# エイリアスでセッションをロード
/sessions load today

# セッション情報を表示
/sessions info today

# エイリアスを削除
/sessions alias --remove today

# すべてのエイリアスをリスト
/sessions aliases
```

## 注記

- セッションは `~/.claude/sessions/` にマークダウンファイルとして保存されます
- エイリアスは `~/.claude/session-aliases.json` に保存されます
- セッション ID は短縮可能です (最初の 4-8 文字で通常は十分ユニークです)
- 頻繁に参照されるセッションにはエイリアスを使用してください
