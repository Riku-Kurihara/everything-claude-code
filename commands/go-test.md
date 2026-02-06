---
description: Go にTDDワークフローを強制します。テーブル駆動テストを最初に書いて、実装します。go test -cover で 80%+ カバレッジを確認します。
---

# Go TDD コマンド

このコマンドは慣用的な Go テストパターンを使用して Go コード用のテスト駆動開発方法論を強制します。

## このコマンドが行うこと

1. **型/インターフェースを定義**: 関数シグネチャを最初にスキャフォルド
2. **テーブル駆動テストを書く**: 包括的なテストケースを作成 (赤)
3. **テストを実行**: テストが正しい理由で失敗することを確認
4. **コードを実装**: 成功するための最小限のコードを書く (緑)
5. **リファクタ**: テストが緑のままで改善
6. **カバレッジをチェック**: 80%+ カバレッジを確認

## いつ使用するか

以下の場合に `/go-test` を使用してください:
- 新しい Go 関数を実装するとき
- 既存コードにテストカバレッジを追加
- バグ修正 (最初に失敗するテストを書く)
- 重要なビジネスロジックを構築
- Go で TDD ワークフローを学ぶ

## TDD サイクル

```
赤     → 失敗するテーブル駆動テストを書く
緑     → 成功するための最小限のコードを実装
リファクタ → コードを改善、テストは緑のままにする
繰り返す  → 次のテストケース
```

## セッション例

````
ユーザー: /go-test メールアドレスを検証する関数が必要です

エージェント:
# TDD セッション: メールバリデーター

## ステップ 1: インターフェースを定義

```go
// validator/email.go
package validator

// ValidateEmail は与えられた文字列が有効なメールアドレスであるかをチェックします。
// 有効な場合は nil を返し、そうでない場合は問題を説明するエラーを返します。
func ValidateEmail(email string) error {
    panic("実装されていない")
}
```

## ステップ 2: テーブル駆動テストを書く (赤)

```go
// validator/email_test.go
package validator

import (
    "testing"
)

func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        // 有効なメール
        {"シンプルなメール", "user@example.com", false},
        {"サブドメイン付き", "user@mail.example.com", false},
        {"プラス記号付き", "user+tag@example.com", false},
        {"ドット付き", "first.last@example.com", false},

        // 無効なメール
        {"空の文字列", "", true},
        {"@記号なし", "userexample.com", true},
        {"ドメインなし", "user@", true},
        {"ローカルパートなし", "@example.com", true},
        {"ダブル@", "user@@example.com", true},
        {"スペース", "user @example.com", true},
        {"TLDなし", "user@example", true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateEmail(tt.email)
            if tt.wantErr && err == nil {
                t.Errorf("ValidateEmail(%q) = nil; エラーが必要", tt.email)
            }
            if !tt.wantErr && err != nil {
                t.Errorf("ValidateEmail(%q) = %v; nil が必要", tt.email, err)
            }
        })
    }
}
```

## ステップ 3: テストを実行 - 失敗を確認

```bash
$ go test ./validator/...

--- FAIL: TestValidateEmail (0.00s)
    --- FAIL: TestValidateEmail/simple_email (0.00s)
        パニック: 実装されていない

失敗
```

✓ テストは予想通り失敗します (パニック)。

## ステップ 4: 最小限のコードを実装 (緑)

```go
// validator/email.go
package validator

import (
    "errors"
    "regexp"
)

var emailRegex = regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)

var (
    ErrEmailEmpty   = errors.New("メールは空にできません")
    ErrEmailInvalid = errors.New("メール形式が無効です")
)

func ValidateEmail(email string) error {
    if email == "" {
        return ErrEmailEmpty
    }
    if !emailRegex.MatchString(email) {
        return ErrEmailInvalid
    }
    return nil
}
```

## ステップ 5: テストを実行 - 成功を確認

```bash
$ go test ./validator/...

成功
ok      project/validator    0.003s
```

✓ すべてのテストが成功しています!

## ステップ 6: カバレッジをチェック

```bash
$ go test -cover ./validator/...

成功
coverage: 100.0% of statements
ok      project/validator    0.003s
```

✓ カバレッジ: 100%

## TDD 完了!
````

## テストパターン

### テーブル駆動テスト
```go
tests := []struct {
    name     string
    input    InputType
    want     OutputType
    wantErr  bool
}{
    {"ケース 1", input1, want1, false},
    {"ケース 2", input2, want2, true},
}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got, err := Function(tt.input)
        // アサーション
    })
}
```

### 並列テスト
```go
for _, tt := range tests {
    tt := tt // キャプチャ
    t.Run(tt.name, func(t *testing.T) {
        t.Parallel()
        // テスト本体
    })
}
```

### テストヘルパー
```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()
    db := createDB()
    t.Cleanup(func() { db.Close() })
    return db
}
```

## カバレッジコマンド

```bash
# 基本的なカバレッジ
go test -cover ./...

# カバレッジプロフィール
go test -coverprofile=coverage.out ./...

# ブラウザで表示
go tool cover -html=coverage.out

# 関数別カバレッジ
go tool cover -func=coverage.out

# 競合状態検出付き
go test -race -cover ./...
```

## カバレッジ目標

| コードタイプ | 目標 |
|-----------|--------|
| 重要なビジネスロジック | 100% |
| パブリック API | 90%+ |
| 一般的なコード | 80%+ |
| 生成されたコード | 除外 |

## TDD ベストプラクティス

**する:**
- テストを最初に書く、実装の前に
- 各変更後にテストを実行
- 包括的なカバレッジのためにテーブル駆動テストを使用
- 実装の詳細ではなく動作をテスト
- エッジケース (空、nil、最大値) を含める

**しない:**
- テストの前に実装を書く
- 赤フェーズをスキップ
- プライベート関数を直接テスト
- テストで `time.Sleep` を使用
- 不安定なテストを無視

## 関連コマンド

- `/go-build` - ビルドエラーを修正
- `/go-review` - 実装後のコードをレビュー
- `/verify` - 完全な検証ループを実行

## 関連

- スキル: `skills/golang-testing/`
- スキル: `skills/tdd-workflow/`
