---
name: iterative-retrieval
description: サブエージェント コンテキスト問題を解決するために、コンテキスト検索を段階的に洗練するパターン
---

# 反復的検索パターン

マルチエージェント ワークフローで、サブエージェントが作業を開始するまでどのコンテキストが必要かわからない「コンテキスト問題」を解決します。

## 問題

サブエージェントは限定的なコンテキストで生成されます。それらは以下を知りません:
- どのファイルに関連するコードが含まれているか
- コードベースに存在するどのパターン
- プロジェクトがどの用語を使用しているか

標準的なアプローチは失敗します:
- **すべてを送信**: コンテキスト制限を超える
- **何も送信しない**: エージェントは重要な情報を欠く
- **必要なものを推測**: しばしば間違っている

## 解決策: 反復的検索

コンテキストを段階的に洗練する 4 フェーズループ:

```
┌─────────────────────────────────────────────┐
│                                             │
│   ┌──────────┐      ┌──────────┐            │
│   │ DISPATCH │─────▶│ EVALUATE │            │
│   └──────────┘      └──────────┘            │
│        ▲                  │                 │
│        │                  ▼                 │
│   ┌──────────┐      ┌──────────┐            │
│   │   LOOP   │◀─────│  REFINE  │            │
│   └──────────┘      └──────────┘            │
│                                             │
│        最大 3 サイクル、その後進める       │
└─────────────────────────────────────────────┘
```

### フェーズ 1: DISPATCH

候補ファイルを収集するための初期の広いクエリ:

```javascript
// 高度な意図から開始
const initialQuery = {
  patterns: ['src/**/*.ts', 'lib/**/*.ts'],
  keywords: ['authentication', 'user', 'session'],
  excludes: ['*.test.ts', '*.spec.ts']
};

// 検索エージェントにディスパッチ
const candidates = await retrieveFiles(initialQuery);
```

### フェーズ 2: EVALUATE

取得されたコンテンツの関連性を評価:

```javascript
function evaluateRelevance(files, task) {
  return files.map(file => ({
    path: file.path,
    relevance: scoreRelevance(file.content, task),
    reason: explainRelevance(file.content, task),
    missingContext: identifyGaps(file.content, task)
  }));
}
```

採点基準:
- **高 (0.8-1.0)**: 目標機能を直接実装
- **中 (0.5-0.7)**: 関連するパターンまたはタイプを含む
- **低 (0.2-0.4)**: 接線的に関連
- **なし (0-0.2)**: 関連なし、除外

### フェーズ 3: REFINE

評価に基づいて検索条件を更新:

```javascript
function refineQuery(evaluation, previousQuery) {
  return {
    // 高い関連性を持つファイルで発見された新しいパターンを追加
    patterns: [...previousQuery.patterns, ...extractPatterns(evaluation)],

    // コードベースで見つかった用語を追加
    keywords: [...previousQuery.keywords, ...extractKeywords(evaluation)],

    // 確認済みの無関係なパスを除外
    excludes: [...previousQuery.excludes, ...evaluation
      .filter(e => e.relevance < 0.2)
      .map(e => e.path)
    ],

    // 特定のギャップに焦点を当てる
    focusAreas: evaluation
      .flatMap(e => e.missingContext)
      .filter(unique)
  };
}
```

### フェーズ 4: LOOP

洗練された基準で繰り返す (最大 3 サイクル):

```javascript
async function iterativeRetrieve(task, maxCycles = 3) {
  let query = createInitialQuery(task);
  let bestContext = [];

  for (let cycle = 0; cycle < maxCycles; cycle++) {
    const candidates = await retrieveFiles(query);
    const evaluation = evaluateRelevance(candidates, task);

    // 十分なコンテキストがあるかをチェック
    const highRelevance = evaluation.filter(e => e.relevance >= 0.7);
    if (highRelevance.length >= 3 && !hasCriticalGaps(evaluation)) {
      return highRelevance;
    }

    // 洗練して続ける
    query = refineQuery(evaluation, query);
    bestContext = mergeContext(bestContext, highRelevance);
  }

  return bestContext;
}
```

## 実践例

### 例 1: バグ修正コンテキスト

```
タスク: "認証トークン有効期限バグを修正"

サイクル 1:
  DISPATCH: src/** で "token"、"auth"、"expiry" を検索
  EVALUATE: auth.ts (0.9)、tokens.ts (0.8)、user.ts (0.3) を発見
  REFINE: "refresh"、"jwt" キーワードを追加; user.ts を除外

サイクル 2:
  DISPATCH: 洗練された用語で検索
  EVALUATE: session-manager.ts (0.95)、jwt-utils.ts (0.85) を発見
  REFINE: 十分なコンテキスト (2 つの高い関連性ファイル)

結果: auth.ts、tokens.ts、session-manager.ts、jwt-utils.ts
```

### 例 2: 機能実装

```
タスク: "API エンドポイントにレート制限を追加"

サイクル 1:
  DISPATCH: routes/** で "rate"、"limit"、"api" を検索
  EVALUATE: マッチなし - コードベースは "throttle" 用語を使用
  REFINE: "throttle"、"middleware" キーワードを追加

サイクル 2:
  DISPATCH: 洗練された用語で検索
  EVALUATE: throttle.ts (0.9)、middleware/index.ts (0.7) を発見
  REFINE: ルーターパターンが必要

サイクル 3:
  DISPATCH: "router"、"express" パターンを検索
  EVALUATE: router-setup.ts (0.8) を発見
  REFINE: 十分なコンテキスト

結果: throttle.ts、middleware/index.ts、router-setup.ts
```

## エージェントとの統合

エージェント プロンプトで使用:

```markdown
このタスクのコンテキストを検索する場合:
1. 広いキーワード検索から開始
2. 各ファイルの関連性を評価 (0-1 スケール)
3. まだどのコンテキストが不足しているかを識別
4. 検索条件を洗練して繰り返す (最大 3 サイクル)
5. 関連性 >= 0.7 のファイルを返す
```

## ベストプラクティス

1. **最初は広く、段階的に狭める** - 初期クエリを過度に指定しないでください
2. **コードベース用語を学習** - 最初のサイクルはしばしば命名規則を明らかにします
3. **何が不足しているかを追跡** - 明示的なギャップ識別が洗練を推進します
4. **「十分」で停止** - 10 個の平凡なものより、3 個の高い関連性ファイル
5. **自信を持って除外** - 低い関連性ファイルは関連性を持つようにはなりません

## 関連項目

- [ロングフォーム ガイド](https://x.com/affaanmustafa/status/2014040193557471352) - サブエージェント オーケストレーション セクション
- `continuous-learning` スキル - 時間とともに改善するパターン用
- エージェント定義 in `~/.claude/agents/`
