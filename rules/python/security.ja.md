# Python セキュリティ

> このファイルは [common/security.md](../common/security.md) を Python 特有の内容で拡張します。

## シークレット管理

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.environ["OPENAI_API_KEY"]  # 欠落している場合は KeyError を発生
```

## セキュリティスキャン

- **bandit** を使用して静的セキュリティ分析：
  ```bash
  bandit -r src/
  ```

## リファレンス

スキルを参照：`django-security` で Django 特有のセキュリティガイドライン（該当する場合）。
