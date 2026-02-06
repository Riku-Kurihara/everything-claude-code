---
name: django-verification
description: Djangoプロジェクト向けの検証ループ: マイグレーション、リンティング、カバレッジ付きテスト、セキュリティスキャン、およびリリースまたはPR前のデプロイ準備チェック。
---

# Django 検証ループ

PRの前に、大きな変更の後に、デプロイ前に実行して、Djangoアプリケーションの品質とセキュリティを確保します。

## フェーズ1: 環境チェック

```bash
# Pythonバージョンを確認
python --version  # プロジェクト要件と一致する必要があります

# 仮想環境を確認
which python
pip list --outdated

# 環境変数を確認
python -c "import os; import environ; print('DJANGO_SECRET_KEY set' if os.environ.get('DJANGO_SECRET_KEY') else 'MISSING: DJANGO_SECRET_KEY')"
```

環境が誤って構成されている場合は、停止して修正してください。

## フェーズ2: コード品質とフォーマッティング

```bash
# 型チェック
mypy . --config-file pyproject.toml

# ruffでリンティング
ruff check . --fix

# blackでフォーマット
black . --check
black .  # 自動修正

# インポートのソート
isort . --check-only
isort .  # 自動修正

# Django固有のチェック
python manage.py check --deploy
```

一般的な問題:
- 公開関数の型ヒント不足
- PEP 8フォーマット違反
- インポート未ソート
- 本番設定に左下されたデバッグ設定

## フェーズ3: マイグレーション

```bash
# 未適用マイグレーションを確認
python manage.py showmigrations

# 欠落マイグレーションを作成
python manage.py makemigrations --check

# マイグレーション適用のドライラン
python manage.py migrate --plan

# マイグレーションを適用（テスト環境）
python manage.py migrate

# マイグレーションの競合をチェック
python manage.py makemigrations --merge  # 競合がある場合のみ
```

レポート:
- 保留中のマイグレーション数
- 任意のマイグレーション競合
- マイグレーションなしのモデル変更

## フェーズ4: テスト + カバレッジ

```bash
# pytestですべてのテストを実行
pytest --cov=apps --cov-report=html --cov-report=term-missing --reuse-db

# 特定のアプリテストを実行
pytest apps/users/tests/

# マーカーで実行
pytest -m "not slow"  # スローテストをスキップ
pytest -m integration  # 統合テストのみ

# カバレッジレポート
open htmlcov/index.html
```

レポート:
- 合計テスト: X成功、Y失敗、Zスキップ
- 全体カバレッジ: XX%
- アプリごとのカバレッジ内訳

カバレッジターゲット:

| コンポーネント | ターゲット |
|------------|--------|
| モデル | 90%+ |
| シリアライザー | 85%+ |
| ビュー | 80%+ |
| サービス | 90%+ |
| 全体 | 80%+ |

## フェーズ5: セキュリティスキャン

```bash
# 依存関係の脆弱性
pip-audit
safety check --full-report

# Djangoセキュリティチェック
python manage.py check --deploy

# Banditセキュリティリンター
bandit -r . -f json -o bandit-report.json

# シークレットスキャン（gitleaksがインストールされている場合）
gitleaks detect --source . --verbose

# 環境変数チェック
python -c "from django.core.exceptions import ImproperlyConfigured; from django.conf import settings; settings.DEBUG"
```

レポート:
- 検出された脆弱性のある依存関係
- セキュリティ設定の問題
- ハードコードされたシークレット検出
- DEBUGモードステータス（本番環境ではFalseであるべき）

## フェーズ6: Django管理コマンド

```bash
# モデルの問題をチェック
python manage.py check

# 静的ファイルを収集
python manage.py collectstatic --noinput --clear

# スーパーユーザーを作成（テストに必要な場合）
echo "from apps.users.models import User; User.objects.create_superuser('admin@example.com', 'admin')" | python manage.py shell

# データベースの整合性
python manage.py check --database default

# キャッシュ検証（Redisを使用している場合）
python -c "from django.core.cache import cache; cache.set('test', 'value', 10); print(cache.get('test'))"
```

## フェーズ7: パフォーマンスチェック

```bash
# Django Debug Toolbarの出力（N+1クエリをチェック）
# DEBUGモード（DEBUG=True）で実行し、ページにアクセス
# SQLパネルで重複クエリを探す

# クエリカウント分析
django-admin debugsqlshell  # django-debug-sqlshellがインストールされている場合

# インデックス不足をチェック
python manage.py shell << EOF
from django.db import connection
with connection.cursor() as cursor:
    cursor.execute("SELECT table_name, index_name FROM information_schema.statistics WHERE table_schema = 'public'")
    print(cursor.fetchall())
EOF
```

レポート:
- ページあたりのクエリ数（通常ページで50未満）
- インデックス不足のデータベース
- 検出された重複クエリ

## フェーズ8: 静的アセット

```bash
# npmの依存関係をチェック（npmを使用している場合）
npm audit
npm audit fix

# 静的ファイルを構築（webpack/viteを使用している場合）
npm run build

# 静的ファイルを確認
ls -la staticfiles/
python manage.py findstatic css/style.css
```

## フェーズ9: 設定レビュー

```python
# Pythonシェルで設定を確認
python manage.py shell << EOF
from django.conf import settings
import os

# 重大チェック
checks = {
    'DEBUG is False': not settings.DEBUG,
    'SECRET_KEY set': bool(settings.SECRET_KEY and len(settings.SECRET_KEY) > 30),
    'ALLOWED_HOSTS set': len(settings.ALLOWED_HOSTS) > 0,
    'HTTPS enabled': getattr(settings, 'SECURE_SSL_REDIRECT', False),
    'HSTS enabled': getattr(settings, 'SECURE_HSTS_SECONDS', 0) > 0,
    'Database configured': settings.DATABASES['default']['ENGINE'] != 'django.db.backends.sqlite3',
}

for check, result in checks.items():
    status = '✓' if result else '✗'
    print(f"{status} {check}")
EOF
```

## フェーズ10: ロギング設定

```bash
# ロギング出力をテスト
python manage.py shell << EOF
import logging
logger = logging.getLogger('django')
logger.warning('Test warning message')
logger.error('Test error message')
EOF

# ログファイルをチェック（設定されている場合）
tail -f /var/log/django/django.log
```

## フェーズ11: API ドキュメント（DRFの場合）

```bash
# スキーマを生成
python manage.py generateschema --format openapi-json > schema.json

# スキーマを検証
# schema.jsonが有効なJSONかをチェック
python -c "import json; json.load(open('schema.json'))"

# Swagger UIにアクセス（drf-yasgを使用している場合）
# ブラウザで http://localhost:8000/swagger/ にアクセス
```

## フェーズ12: Diffレビュー

```bash
# Diffの統計情報を表示
git diff --stat

# 実際の変更を表示
git diff

# 変更されたファイルを表示
git diff --name-only

# 一般的な問題をチェック
git diff | grep -i "todo\|fixme\|hack\|xxx"
git diff | grep "print("  # デバッグステートメント
git diff | grep "DEBUG = True"  # デバッグモード
git diff | grep "import pdb"  # デバッガー
```

チェックリスト:
- デバッグステートメント（print、pdb、breakpoint()）なし
- 重大コードのTODO/FIXMEコメントなし
- ハードコードされたシークレットまたは認証情報なし
- モデル変更のためのデータベースマイグレーション含む
- 設定変更は文書化されている
- 外部呼び出しにはエラーハンドリングがある
- トランザクション管理が必要な場所にはある

## 出力テンプレート

```
DJANGO VERIFICATION REPORT
==========================

フェーズ1: 環境チェック
  ✓ Python 3.11.5
  ✓ 仮想環境がアクティブ
  ✓ すべての環境変数が設定

フェーズ2: コード品質
  ✓ mypy: 型エラーなし
  ✗ ruff: 3つの問題が見つかった（自動修正）
  ✓ black: フォーマットの問題なし
  ✓ isort: インポートは正しくソートされている
  ✓ manage.py check: 問題なし

フェーズ3: マイグレーション
  ✓ 未適用マイグレーションなし
  ✓ マイグレーション競合なし
  ✓ すべてのモデルにマイグレーションがある

フェーズ4: テスト + カバレッジ
  テスト: 247成功、0失敗、5スキップ
  カバレッジ:
    全体: 87%
    users: 92%
    products: 89%
    orders: 85%
    payments: 91%

フェーズ5: セキュリティスキャン
  ✗ pip-audit: 2つの脆弱性が見つかった（修正が必要）
  ✓ safety check: 問題なし
  ✓ bandit: セキュリティ問題なし
  ✓ シークレット検出なし
  ✓ DEBUG = False

フェーズ6: Djangoコマンド
  ✓ collectstatic完了
  ✓ データベースの整合性OK
  ✓ キャッシュバックエンドに到達可能

フェーズ7: パフォーマンス
  ✓ N+1クエリ検出なし
  ✓ データベースインデックスが設定されている
  ✓ クエリカウントは許容範囲内

フェーズ8: 静的アセット
  ✓ npm audit: 脆弱性なし
  ✓ アセットが正常に構築された
  ✓ 静的ファイルが収集されている

フェーズ9: 設定
  ✓ DEBUG = False
  ✓ SECRET_KEYが設定
  ✓ ALLOWED_HOSTSが設定
  ✓ HTTPS有効化
  ✓ HSTS有効化
  ✓ データベースが設定

フェーズ10: ロギング
  ✓ ロギングが設定
  ✓ ログファイルは書き込み可能

フェーズ11: API ドキュメント
  ✓ スキーマが生成
  ✓ Swagger UIにアクセス可能

フェーズ12: Diffレビュー
  ファイル変更: 12
  +450, -120 行
  ✓ デバッグステートメントなし
  ✓ ハードコードされたシークレットなし
  ✓ マイグレーション含む

推奨事項: ⚠️ デプロイ前にpip-auditの脆弱性を修正

次のステップ:
1. 脆弱性のある依存関係を更新
2. セキュリティスキャンを再実行
3. ステージング環境にデプロイして最終テスト
```

## デプロイ前チェックリスト

- [ ] すべてのテストが成功
- [ ] カバレッジ >= 80%
- [ ] セキュリティ脆弱性なし
- [ ] 未適用マイグレーションなし
- [ ] 本番設定で DEBUG = False
- [ ] SECRET_KEYが正しく設定
- [ ] ALLOWED_HOSTSが正しく設定
- [ ] データベースバックアップが有効化
- [ ] 静的ファイルが収集され提供
- [ ] ロギングが設定され機能
- [ ] エラー監視（Sentry等）が設定
- [ ] CDNが設定（該当する場合）
- [ ] Redis/キャッシュバックエンドが設定
- [ ] Celeryワーカーが実行中（該当する場合）
- [ ] HTTPS/SSLが設定
- [ ] 環境変数が文書化

## 継続統合

### GitHub Actions例

```yaml
# .github/workflows/django-verification.yml
name: Django Verification

on: [push, pull_request]

jobs:
  verify:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Cache pip
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install ruff black mypy pytest pytest-django pytest-cov bandit safety pip-audit

      - name: Code quality checks
        run: |
          ruff check .
          black . --check
          isort . --check-only
          mypy .

      - name: Security scan
        run: |
          bandit -r . -f json -o bandit-report.json
          safety check --full-report
          pip-audit

      - name: Run tests
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
          DJANGO_SECRET_KEY: test-secret-key
        run: |
          pytest --cov=apps --cov-report=xml --cov-report=term-missing

      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

## クイックリファレンス

| チェック | コマンド |
|--------|---------|
| 環境 | `python --version` |
| 型チェック | `mypy .` |
| リンティング | `ruff check .` |
| フォーマット | `black . --check` |
| マイグレーション | `python manage.py makemigrations --check` |
| テスト | `pytest --cov=apps` |
| セキュリティ | `pip-audit && bandit -r .` |
| Django check | `python manage.py check --deploy` |
| Collectstatic | `python manage.py collectstatic --noinput` |
| Diff統計 | `git diff --stat` |

覚えておくこと: 自動検証は一般的な問題をキャッチしますが、手動コードレビューとステージング環境でのテストに代わるものではありません。
