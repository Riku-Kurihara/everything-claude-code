---
name: postgres-patterns
description: クエリ最適化、スキーマ設計、インデックス作成、セキュリティに関する PostgreSQL データベース パターン。Supabase ベストプラクティスに基づいています。
---

# PostgreSQL パターン

PostgreSQL ベストプラクティスのクイックリファレンス。詳細なガイダンスについては、`database-reviewer` エージェントを使用してください。

## アクティベーション時

- SQL クエリまたはマイグレーションを書く場合
- データベース スキーマを設計する場合
- クエリの低速化を解決する場合
- 行レベルセキュリティを実装する場合
- コネクション プーリングを設定する場合

## クイックリファレンス

### インデックスチートシート

| クエリパターン | インデックスタイプ | 例 |
|--------------|------------|---------|
| `WHERE col = value` | B-tree (デフォルト) | `CREATE INDEX idx ON t (col)` |
| `WHERE col > value` | B-tree | `CREATE INDEX idx ON t (col)` |
| `WHERE a = x AND b > y` | 複合 | `CREATE INDEX idx ON t (a, b)` |
| `WHERE jsonb @> '{}'` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| `WHERE tsv @@ query` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| 時系列範囲 | BRIN | `CREATE INDEX idx ON t USING brin (col)` |

### データ型クイックリファレンス

| ユースケース | 正しい型 | 避けるべき |
|----------|-------------|-------|
| ID | `bigint` | `int`、ランダムな UUID |
| 文字列 | `text` | `varchar(255)` |
| タイムスタンプ | `timestamptz` | `timestamp` |
| 金額 | `numeric(10,2)` | `float` |
| フラグ | `boolean` | `varchar`、`int` |

### 共通パターン

**複合インデックス順序:**
```sql
-- 等号列を最初に、次に範囲列
CREATE INDEX idx ON orders (status, created_at);
-- これに対して機能: WHERE status = 'pending' AND created_at > '2024-01-01'
```

**カバーインデックス:**
```sql
CREATE INDEX idx ON users (email) INCLUDE (name, created_at);
-- SELECT email, name, created_at のテーブル検索を回避
```

**部分インデックス:**
```sql
CREATE INDEX idx ON users (email) WHERE deleted_at IS NULL;
-- より小さいインデックス、アクティブユーザーのみを含む
```

**RLS ポリシー (最適化):**
```sql
CREATE POLICY policy ON orders
  USING ((SELECT auth.uid()) = user_id);  -- SELECT でラップ!
```

**UPSERT:**
```sql
INSERT INTO settings (user_id, key, value)
VALUES (123, 'theme', 'dark')
ON CONFLICT (user_id, key)
DO UPDATE SET value = EXCLUDED.value;
```

**カーソル ページネーション:**
```sql
SELECT * FROM products WHERE id > $last_id ORDER BY id LIMIT 20;
-- OFFSET は O(n) なのに対して O(1)
```

**キュー処理:**
```sql
UPDATE jobs SET status = 'processing'
WHERE id = (
  SELECT id FROM jobs WHERE status = 'pending'
  ORDER BY created_at LIMIT 1
  FOR UPDATE SKIP LOCKED
) RETURNING *;
```

### アンチパターン検出

```sql
-- インデックスがない外部キーを検索
SELECT conrelid::regclass, a.attname
FROM pg_constraint c
JOIN pg_attribute a ON a.attrelid = c.conrelid AND a.attnum = ANY(c.conkey)
WHERE c.contype = 'f'
  AND NOT EXISTS (
    SELECT 1 FROM pg_index i
    WHERE i.indrelid = c.conrelid AND a.attnum = ANY(i.indkey)
  );

-- 低速なクエリを検索
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
WHERE mean_exec_time > 100
ORDER BY mean_exec_time DESC;

-- テーブル ブロートをチェック
SELECT relname, n_dead_tup, last_vacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;
```

### 設定テンプレート

```sql
-- コネクション制限 (RAM に応じて調整)
ALTER SYSTEM SET max_connections = 100;
ALTER SYSTEM SET work_mem = '8MB';

-- タイムアウト
ALTER SYSTEM SET idle_in_transaction_session_timeout = '30s';
ALTER SYSTEM SET statement_timeout = '30s';

-- モニタリング
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- セキュリティデフォルト
REVOKE ALL ON SCHEMA public FROM public;

SELECT pg_reload_conf();
```

## 関連項目

- エージェント: `database-reviewer` - 完全なデータベースレビュー ワークフロー
- スキル: `clickhouse-io` - ClickHouse 分析パターン
- スキル: `backend-patterns` - API およびバックエンド パターン

---

*Based on [Supabase Agent Skills](https://github.com/supabase/agent-skills) (MIT License)*
