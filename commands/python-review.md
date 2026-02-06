---
description: PEP 8 コンプライアンス、型ヒント、セキュリティ、Pythonic イディオムのための包括的な Python コードレビュー。python-reviewer エージェントを呼び出します。
---

# Python コードレビュー

このコマンドは **python-reviewer** エージェントを呼び出して、Python固有の包括的なコードレビューを実行します。

## このコマンドが行うこと

1. **Python 変更を特定**: `git diff` で変更された `.py` ファイルを検索
2. **静的分析を実行**: `ruff`, `mypy`, `pylint`, `black --check` を実行
3. **セキュリティスキャン**: SQLインジェクション、コマンドインジェクション、安全でないデシリアライゼーションをチェック
4. **型安全性レビュー**: 型ヒントと mypy エラーを分析
5. **Pythonic コードチェック**: コードが PEP 8 と Python ベストプラクティスに従っていることを確認
6. **レポート生成**: 重要度別に問題を分類

## いつ使用するか

以下の場合に `/python-review` を使用してください:
- Python コードを書いたり変更した後
- Python 変更をコミットする前
- Python コードを含むプルリクエストをレビューするとき
- 新しい Python コードベースへのオンボーディング時
- Pythonic パターンとイディオムを学ぶときに

## レビューカテゴリー

### 重大 (修正する必要がある)
- SQLインジェクション/コマンドインジェクション脆弱性
- 安全でない eval/exec の使用法
- Pickle の安全でないデシリアライゼーション
- ハードコードされたクレデンシャル
- YAML の安全でないロード
- エラーを隠すむき出しの except 句

### 高い (修正すべき)
- パブリック関数の型ヒントの欠落
- ミューテーション可能なデフォルト引数
- 例外を静かに飲み込む
- リソース用のコンテキストマネージャーを使用していない
- 内包表記の代わりに C スタイルループ
- isinstance() の代わりに type() を使用
- ロックなしの競合状態

### MEDIUM (Consider)
- PEP 8 formatting violations
- Missing docstrings on public functions
- Print statements instead of logging
- Inefficient string operations
- Magic numbers without named constants
- Not using f-strings for formatting
- Unnecessary list creation

## 実行される自動チェック

```bash
# 型チェック
mypy .

# リントとフォーマット
ruff check .
black --check .
isort --check-only .

# セキュリティスキャン
bandit -r .

# 依存性監査
pip-audit
safety check

# テスト
pytest --cov=app --cov-report=term-missing
```

## 使用例

```text
ユーザー: /python-review

エージェント:
# Python コードレビューレポート

## レビュー済みファイル
- app/routes/user.py (変更)
- app/services/auth.py (変更)

## 静的分析結果
✓ ruff: 問題なし
✓ mypy: エラーなし
⚠️ black: 2 ファイルがフォーマット必要
✓ bandit: セキュリティ問題なし

## 見つかった問題

[重大] SQL インジェクション脆弱性
ファイル: app/routes/user.py:42
問題: ユーザー入力が SQL クエリに直接挿入されている
```python
query = f"SELECT * FROM users WHERE id = {user_id}"  # 悪い
```
修正: パラメータ化されたクエリを使用
```python
query = "SELECT * FROM users WHERE id = %s"  # 良い
cursor.execute(query, (user_id,))
```

[高い] ミューテーション可能なデフォルト引数
ファイル: app/services/auth.py:18
問題: ミューテーション可能なデフォルト引数は共有状態を引き起こす
```python
def process_items(items=[]):  # 悪い
    items.append("new")
    return items
```
修正: デフォルトとして None を使用
```python
def process_items(items=None):  # 良い
    if items is None:
        items = []
    items.append("new")
    return items
```

[中程度] 型ヒントの欠落
ファイル: app/services/auth.py:25
問題: 型アノテーションなしのパブリック関数
```python
def get_user(user_id):  # 悪い
    return db.find(user_id)
```
修正: 型ヒントを追加
```python
def get_user(user_id: str) -> Optional[User]:  # 良い
    return db.find(user_id)
```

[中程度] コンテキストマネージャーを使用していない
ファイル: app/routes/user.py:55
問題: 例外でファイルが閉じられない
```python
f = open("config.json")  # 悪い
data = f.read()
f.close()
```
修正: コンテキストマネージャーを使用
```python
with open("config.json") as f:  # 良い
    data = f.read()
```

## サマリー
- 重大: 1
- 高い: 1
- 中程度: 2

推奨: ❌ 重大な問題が修正されるまでマージをブロック

## フォーマット必須
実行: `black app/routes/user.py app/services/auth.py`
```

## 承認基準

| ステータス | 条件 |
|--------|-----------|
| ✅ 承認 | 重大または高い問題なし |
| ⚠️ 警告 | 中程度の問題のみ (注意してマージ) |
| ❌ ブロック | 重大または高い問題が見つかった |

## 他のコマンドとの統合

- テストが成功することを確認するために `/python-test` を最初に実行
- Python 固有以外の懸念については `/code-review` を使用
- コミット前に `/python-review` を実行
- 静的分析ツールが失敗した場合は `/build-fix` を使用

## フレームワーク固有のレビュー

### Django プロジェクト
レビューは以下をチェック:
- N+1 クエリ問題 (`select_related` と `prefetch_related` を使用)
- モデル変更用の欠落マイグレーション
- ORM が機能する場合の生SQL使用法
- マルチステップ操作用の欠落 `transaction.atomic()`

### FastAPI プロジェクト
レビューは以下をチェック:
- CORS 誤設定
- リクエスト検証用の Pydantic モデル
- レスポンスモデルの正確性
- 適切な async/await 使用法
- 依存性注入パターン

### Flask プロジェクト
レビューは以下をチェック:
- コンテキスト管理 (アプリコンテキスト、リクエストコンテキスト)
- 適切なエラーハンドリング
- ブループリント組織
- 設定管理

## 関連

- エージェント: `agents/python-reviewer.md`
- スキル: `skills/python-patterns/`, `skills/python-testing/`

## 一般的な修正

### 型ヒントを追加
```python
# 前
def calculate(x, y):
    return x + y

# 後
from typing import Union

def calculate(x: Union[int, float], y: Union[int, float]) -> Union[int, float]:
    return x + y
```

### コンテキストマネージャーを使用
```python
# 前
f = open("file.txt")
data = f.read()
f.close()

# 後
with open("file.txt") as f:
    data = f.read()
```

### リスト内包表記を使用
```python
# 前
result = []
for item in items:
    if item.active:
        result.append(item.name)

# 後
result = [item.name for item in items if item.active]
```

### ミューテーション可能なデフォルトを修正
```python
# 前
def append(value, items=[]):
    items.append(value)
    return items

# 後
def append(value, items=None):
    if items is None:
        items = []
    items.append(value)
    return items
```

### f-strings を使用 (Python 3.6+)
```python
# 前
name = "Alice"
greeting = "Hello, " + name + "!"
greeting2 = "Hello, {}".format(name)

# 後
greeting = f"Hello, {name}!"
```

### ループ内での文字列連結を修正
```python
# 前
result = ""
for item in items:
    result += str(item)

# 後
result = "".join(str(item) for item in items)
```

## Python バージョン互換性

レビューアーは、より新しい Python バージョンの機能を使用するコードに注記します:

| 機能 | 最小 Python |
|---------|----------------|
| 型ヒント | 3.5+ |
| f-strings | 3.6+ |
| ウォルス演算子 (`:=`) | 3.8+ |
| 位置専用パラメーター | 3.8+ |
| Match ステートメント | 3.10+ |
| 型ユニオン (`x &#124; None`) | 3.10+ |

プロジェクトの `pyproject.toml` または `setup.py` が正しい最小 Python バージョンを指定していることを確認してください。
