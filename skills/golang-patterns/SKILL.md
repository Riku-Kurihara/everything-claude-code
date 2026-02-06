---
name: golang-patterns
description: 慣用的なGoパターン、ベストプラクティス、およびロバスト、効率的で保守性の高いGoアプリケーションを構築するための慣例。
---

# Go開発パターン

ロバスト、効率的で保守性の高いアプリケーションを構築するための慣用的なGoパターンとベストプラクティス。

## アクティベーション時

- 新しいGoコードを書く場合
- Goコードをレビューする場合
- 既存のGoコードをリファクタリングする場合
- Goパッケージ/モジュールを設計する場合

## 基本原則

### 1. シンプリティと明確性

Goはシンプリティを優先します。コードは明白で読みやすいべきです。

```go
// 良い例: 明確で直感的
func GetUser(id string) (*User, error) {
    user, err := db.FindUser(id)
    if err != nil {
        return nil, fmt.Errorf("get user %s: %w", id, err)
    }
    return user, nil
}

// 悪い例: 過度に複雑
func GetUser(id string) (*User, error) {
    return func() (*User, error) {
        if u, e := db.FindUser(id); e == nil {
            return u, nil
        } else {
            return nil, e
        }
    }()
}
```

### 2. ゼロ値を有用にする

型の設計時に、初期化せずにゼロ値がすぐに使用できるようにします。

```go
// 良い例: ゼロ値が有用
type Counter struct {
    mu    sync.Mutex
    count int // ゼロ値は0で、使用可能
}

func (c *Counter) Inc() {
    c.mu.Lock()
    c.count++
    c.mu.Unlock()
}

// 良い例: bytes.Bufferはゼロ値で動作
var buf bytes.Buffer
buf.WriteString("hello")

// 悪い例: 初期化が必要
type BadCounter struct {
    counts map[string]int // nilマップはパニック
}
```

### 3. インターフェースを受け取り、構造体を返す

関数はインターフェースパラメータを受け取り、具体的な型を返すべきです。

```go
// 良い例: インターフェースを受け取り、具体的な型を返す
func ProcessData(r io.Reader) (*Result, error) {
    data, err := io.ReadAll(r)
    if err != nil {
        return nil, err
    }
    return &Result{Data: data}, nil
}

// 悪い例: インターフェースを返す（実装詳細を不必要に隠す）
func ProcessData(r io.Reader) (io.Reader, error) {
    // ...
}
```

## エラー処理パターン

### コンテキスト付きエラーラッピング

```go
// 良い例: コンテキストと共にエラーをラップ
func LoadConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("load config %s: %w", path, err)
    }

    var cfg Config
    if err := json.Unmarshal(data, &cfg); err != nil {
        return nil, fmt.Errorf("parse config %s: %w", path, err)
    }

    return &cfg, nil
}
```

### カスタムエラー型

```go
// ドメイン固有のエラーを定義
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
}

// 共通の場合のセンチネルエラー
var (
    ErrNotFound     = errors.New("resource not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrInvalidInput = errors.New("invalid input")
)
```

### errors.Isとerrors.Asでのエラーチェック

```go
func HandleError(err error) {
    // 特定のエラーをチェック
    if errors.Is(err, sql.ErrNoRows) {
        log.Println("No records found")
        return
    }

    // エラー型をチェック
    var validationErr *ValidationError
    if errors.As(err, &validationErr) {
        log.Printf("Validation error on field %s: %s",
            validationErr.Field, validationErr.Message)
        return
    }

    // 未知のエラー
    log.Printf("Unexpected error: %v", err)
}
```

### エラーを無視しない

```go
// 悪い例: ブランク識別子でエラーを無視
result, _ := doSomething()

// 良い例: 処理するか、なぜ無視するか明確に
result, err := doSomething()
if err != nil {
    return err
}

// 受け入れられる: エラーが本当に重要でない場合（稀）
_ = writer.Close() // ベストエフォート クリーンアップ、エラーは他で記録
```

## 並行処理パターン

### ワーカープール

```go
func WorkerPool(jobs <-chan Job, results chan<- Result, numWorkers int) {
    var wg sync.WaitGroup

    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                results <- process(job)
            }
        }()
    }

    wg.Wait()
    close(results)
}
```

### キャンセルとタイムアウト用のContext

```go
func FetchWithTimeout(ctx context.Context, url string) ([]byte, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()

    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, fmt.Errorf("create request: %w", err)
    }

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, fmt.Errorf("fetch %s: %w", url, err)
    }
    defer resp.Body.Close()

    return io.ReadAll(resp.Body)
}
```

### グレースフルシャットダウン

```go
func GracefulShutdown(server *http.Server) {
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

    <-quit
    log.Println("Shutting down server...")

    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := server.Shutdown(ctx); err != nil {
        log.Fatalf("Server forced to shutdown: %v", err)
    }

    log.Println("Server exited")
}
```

### coordinated Goroutines用のerrgroup

```go
import "golang.org/x/sync/errgroup"

func FetchAll(ctx context.Context, urls []string) ([][]byte, error) {
    g, ctx := errgroup.WithContext(ctx)
    results := make([][]byte, len(urls))

    for i, url := range urls {
        i, url := i, url // ループ変数をキャプチャ
        g.Go(func() error {
            data, err := FetchWithTimeout(ctx, url)
            if err != nil {
                return err
            }
            results[i] = data
            return nil
        })
    }

    if err := g.Wait(); err != nil {
        return nil, err
    }
    return results, nil
}
```

### Goroutineリークの回避

```go
// 悪い例: Contextがキャンセルされるとゴルーチンリーク
func leakyFetch(ctx context.Context, url string) <-chan []byte {
    ch := make(chan []byte)
    go func() {
        data, _ := fetch(url)
        ch <- data // レシーバーがない場合永遠にブロック
    }()
    return ch
}

// 良い例: キャンセルを正しく処理
func safeFetch(ctx context.Context, url string) <-chan []byte {
    ch := make(chan []byte, 1) // バッファ付きチャネル
    go func() {
        data, err := fetch(url)
        if err != nil {
            return
        }
        select {
        case ch <- data:
        case <-ctx.Done():
        }
    }()
    return ch
}
```

## インターフェース設計

### 小さく、フォーカスされたインターフェース

```go
// 良い例: シングルメソッドインターフェース
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

// 必要に応じてインターフェースを合成
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

### インターフェースを使用する場所で定義する

```go
// コンシューマパッケージで定義、プロバイダではない
package service

// UserStoreは、このサービスが何を必要としているかを定義
type UserStore interface {
    GetUser(id string) (*User, error)
    SaveUser(user *User) error
}

type Service struct {
    store UserStore
}

// 具体的な実装は別のパッケージに入ることができます
// このインターフェースについて知る必要はありません
```

### 型アサーションを使用したオプション動作

```go
type Flusher interface {
    Flush() error
}

func WriteAndFlush(w io.Writer, data []byte) error {
    if _, err := w.Write(data); err != nil {
        return err
    }

    // サポートされている場合はFlush
    if f, ok := w.(Flusher); ok {
        return f.Flush()
    }
    return nil
}
```

## パッケージ構成

### 標準プロジェクトレイアウト

```text
myproject/
├── cmd/
│   └── myapp/
│       └── main.go           # エントリポイント
├── internal/
│   ├── handler/              # HTTPハンドラー
│   ├── service/              # ビジネスロジック
│   ├── repository/           # データアクセス
│   └── config/               # 設定
├── pkg/
│   └── client/               # パブリック API クライアント
├── api/
│   └── v1/                   # API定義 (proto, OpenAPI)
├── testdata/                 # テストフィクスチャ
├── go.mod
├── go.sum
└── Makefile
```

### パッケージの命名

```go
// 良い例: 短い、小文字、アンダースコアなし
package http
package json
package user

// 悪い例: 冗長、混合ケース、または冗長
package httpHandler
package json_parser
package userService // 冗長な'Service'サフィックス
```

### パッケージレベルの状態を避ける

```go
// 悪い例: グローバル可変状態
var db *sql.DB

func init() {
    db, _ = sql.Open("postgres", os.Getenv("DATABASE_URL"))
}

// 良い例: 依存性の注入
type Server struct {
    db *sql.DB
}

func NewServer(db *sql.DB) *Server {
    return &Server{db: db}
}
```

## 構造体設計

### 関数型オプションパターン

```go
type Server struct {
    addr    string
    timeout time.Duration
    logger  *log.Logger
}

type Option func(*Server)

func WithTimeout(d time.Duration) Option {
    return func(s *Server) {
        s.timeout = d
    }
}

func WithLogger(l *log.Logger) Option {
    return func(s *Server) {
        s.logger = l
    }
}

func NewServer(addr string, opts ...Option) *Server {
    s := &Server{
        addr:    addr,
        timeout: 30 * time.Second, // デフォルト
        logger:  log.Default(),    // デフォルト
    }
    for _, opt := range opts {
        opt(s)
    }
    return s
}

// 使用例
server := NewServer(":8080",
    WithTimeout(60*time.Second),
    WithLogger(customLogger),
)
```

### 合成のための埋め込み

```go
type Logger struct {
    prefix string
}

func (l *Logger) Log(msg string) {
    fmt.Printf("[%s] %s\n", l.prefix, msg)
}

type Server struct {
    *Logger // 埋め込み - ServerはLogメソッドを取得
    addr    string
}

func NewServer(addr string) *Server {
    return &Server{
        Logger: &Logger{prefix: "SERVER"},
        addr:   addr,
    }
}

// 使用例
s := NewServer(":8080")
s.Log("Starting...") // 埋め込みLogger.Logを呼び出す
```

## メモリとパフォーマンス

### サイズが既知の場合、スライスを事前割り当て

```go
// 悪い例: スライスが複数回成長
func processItems(items []Item) []Result {
    var results []Result
    for _, item := range items {
        results = append(results, process(item))
    }
    return results
}

// 良い例: 単一の割り当て
func processItems(items []Item) []Result {
    results := make([]Result, 0, len(items))
    for _, item := range items {
        results = append(results, process(item))
    }
    return results
}
```

### 頻繁な割り当てにはsync.Poolを使用

```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func ProcessRequest(data []byte) []byte {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer func() {
        buf.Reset()
        bufferPool.Put(buf)
    }()

    buf.Write(data)
    // 処理...
    return buf.Bytes()
}
```

### ループ内での文字列連結を避ける

```go
// 悪い例: 多くの文字列割り当てを作成
func join(parts []string) string {
    var result string
    for _, p := range parts {
        result += p + ","
    }
    return result
}

// 良い例: strings.Builderで単一割り当て
func join(parts []string) string {
    var sb strings.Builder
    for i, p := range parts {
        if i > 0 {
            sb.WriteString(",")
        }
        sb.WriteString(p)
    }
    return sb.String()
}

// ベスト: 標準ライブラリを使用
func join(parts []string) string {
    return strings.Join(parts, ",")
}
```

## Goツール統合

### 必須コマンド

```bash
# ビルドと実行
go build ./...
go run ./cmd/myapp

# テスト
go test ./...
go test -race ./...
go test -cover ./...

# 静的分析
go vet ./...
staticcheck ./...
golangci-lint run

# モジュール管理
go mod tidy
go mod verify

# フォーマット
gofmt -w .
goimports -w .
```

### 推奨Linter設定 (.golangci.yml)

```yaml
linters:
  enable:
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - unused
    - gofmt
    - goimports
    - misspell
    - unconvert
    - unparam

linters-settings:
  errcheck:
    check-type-assertions: true
  govet:
    check-shadowing: true

issues:
  exclude-use-default: false
```

## クイックリファレンス: Go慣用句

| 慣用句 | 説明 |
|-------|------|
| インターフェースを受け取り、構造体を返す | 関数がインターフェース パラメータを受け取り、具体的な型を返す |
| エラーは値 | エラーを第一級の値として扱う、例外ではない |
| メモリ共有によって通信しない | ゴルーチン間の調整にはチャネルを使用 |
| ゼロ値を有用にする | 型は明示的な初期化なしで機能すべき |
| 少しのコピーは少しの依存より良い | 不要な外部依存を避ける |
| 明確はクレバーより良い | 賢さより読みやすさを優先 |
| gofmtは誰のお気に入りでもないが、皆の友 | 常にgofmt/goimportsでフォーマット |
| 早期リターン | エラーを最初に処理し、ハッピーパスをインデント外に保つ |

## 避けるべきアンチパターン

```go
// 悪い例: 長い関数での裸のリターン
func process() (result int, err error) {
    // ... 50行 ...
    return // 何が返されているか？
}

// 悪い例: パニックを制御フローに使用
func GetUser(id string) *User {
    user, err := db.Find(id)
    if err != nil {
        panic(err) // これをしない
    }
    return user
}

// 悪い例: 構造体にcontextを渡す
type Request struct {
    ctx context.Context // Contextは最初のパラメータにすべき
    ID  string
}

// 良い例: 最初のパラメータとしてContext
func ProcessRequest(ctx context.Context, id string) error {
    // ...
}

// 悪い例: 値と ポインターレシーバを混合
type Counter struct{ n int }
func (c Counter) Value() int { return c.n }    // 値レシーバ
func (c *Counter) Increment() { c.n++ }        // ポインターレシーバ
// 1つのスタイルを選んで一貫性を保つ
```

**注意**: Goコードは最良の意味で退屈であるべき - 予測可能で、一貫性があり、理解しやすい。確信がない場合は、シンプルに保ちます。
