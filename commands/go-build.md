---
description: Go のビルドエラー、go vet の警告、リンターの問題を段階的に修正します。最小限の修正のために go-build-resolver エージェントを呼び出します。
---

# Go ビルドと修正

このコマンドは **go-build-resolver** エージェントを呼び出して、Go ビルドエラーを最小限の変更で段階的に修正します。

## このコマンドが行うこと

1. **診断を実行**: `go build`, `go vet`, `staticcheck` を実行
2. **エラーを解析**: ファイル単位でグループ化して重要度でソート
3. **段階的に修正**: 一度に1つのエラーを修正
4. **各修正を検証**: 変更後にビルドを再実行
5. **サマリーを報告**: 修正されたものと残っているものを表示

## いつ使用するか

以下の場合に `/go-build` を使用してください:
- `go build ./...` がエラーで失敗
- `go vet ./...` が問題を報告
- `golangci-lint run` が警告を表示
- モジュール依存性が壊れている
- ビルドを壊す変更をプル後

## 実行される診断コマンド

```bash
# 主要なビルドチェック
go build ./...

# 静的分析
go vet ./...

# 拡張リント (利用可能な場合)
staticcheck ./...
golangci-lint run

# モジュール問題
go mod verify
go mod tidy -v
```

## セッション例

```text
ユーザー: /go-build

エージェント:
# Go ビルド解決

## 初期診断

```bash
$ go build ./...
internal/service/user.go:25:15: undefined: UserRepository
internal/handler/api.go:42:9: cannot use x (type string) as type int
internal/handler/api.go:58:2: missing return at end of function
```

見つかったエラー: 3

## 修正 1: 未定義の識別子

ファイル: internal/service/user.go:25
エラー: undefined: UserRepository
原因: インポートの欠落

```go
// インポートを追加
import "project/internal/repository"

// 変更
var repo UserRepository
// 次へ
var repo repository.UserRepository
```

```bash
$ go build ./...
# 残り 2 エラー
```

## 修正 2: 型の不一致

ファイル: internal/handler/api.go:42
エラー: cannot use x (type string) as type int

```go
// 変更
count := params.Get("count")
// 次へ
countStr := params.Get("count")
count, _ := strconv.Atoi(countStr)
```

```bash
$ go build ./...
# 残り 1 エラー
```

## 修正 3: 戻り値の欠落

ファイル: internal/handler/api.go:58
エラー: missing return at end of function

```go
func GetUser(id string) (*User, error) {
    if id == "" {
        return nil, ErrInvalidID
    }
    user := findUser(id)
    // 欠落していた戻り値を追加
    return user, nil
}
```

```bash
$ go build ./...
# ビルド成功!
```

## 最終検証

```bash
$ go vet ./...
# 問題なし

$ go test ./...
ok      project/internal/service   0.015s
ok      project/internal/handler   0.023s
```

## サマリー

| メトリクス | 数 |
|--------|-------|
| 修正されたビルドエラー | 3 |
| 修正された Vet 警告 | 0 |
| 変更されたファイル | 2 |
| 残っている問題 | 0 |

ビルドステータス: ✅ 成功
```

## 修正された一般的なエラー

| エラー | 典型的な修正 |
|-------|-------------|
| `undefined: X` | インポートを追加またはタイプミスを修正 |
| `cannot use X as Y` | 型変換または割り当てを修正 |
| `missing return` | return ステートメントを追加 |
| `X does not implement Y` | 欠落しているメソッドを追加 |
| `import cycle` | パッケージを再構成 |
| `declared but not used` | 変数を削除または使用 |
| `cannot find package` | `go get` または `go mod tidy` |

## 修正戦略

1. **ビルドエラーを最初に修正** - コードはコンパイルする必要があります
2. **Vet 警告を次に修正** - 疑わしい構造を修正
3. **リント警告を 3 番目に修正** - スタイルとベストプラクティス
4. **一度に 1 つの修正** - 各変更を確認
5. **最小限の変更** - リファクタリングではなく修正のみ

## 停止条件

エージェントは以下の場合は停止して報告します:
- 3 回の試行後も同じエラーが続く
- 修正がより多くのエラーを導入
- アーキテクチャ変更が必要
- 外部依存性の欠落

## 関連コマンド

- `/go-test` - ビルド成功後にテストを実行
- `/go-review` - コード品質をレビュー
- `/verify` - 完全な検証ループ

## 関連

- エージェント: `agents/go-build-resolver.md`
- スキル: `skills/golang-patterns/`
