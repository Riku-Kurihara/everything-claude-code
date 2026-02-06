# Python パターン

> このファイルは [common/patterns.md](../common/patterns.md) を Python 特有の内容で拡張します。

## Protocol（ダック型）

```python
from typing import Protocol

class Repository(Protocol):
    def find_by_id(self, id: str) -> dict | None: ...
    def save(self, entity: dict) -> dict: ...
```

## DTO としてのデータクラス

```python
from dataclasses import dataclass

@dataclass
class CreateUserRequest:
    name: str
    email: str
    age: int | None = None
```

## コンテキストマネージャーとジェネレーター

- コンテキストマネージャー（`with` ステートメント）をリソース管理に使用
- 遅延評価とメモリ効率的な反復にジェネレーターを使用

## リファレンス

スキルを参照：`python-patterns` でデコレーター、並行処理、パッケージ構成を含む包括的なパターン。
