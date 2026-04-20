# Go Patterns Reference

## Concurrency Patterns

### Worker Pool with errgroup
```go
import "golang.org/x/sync/errgroup"

func processAll(ctx context.Context, items []Item) error {
    g, ctx := errgroup.WithContext(ctx)
    sem := make(chan struct{}, 8) // bounded concurrency

    for _, item := range items {
        item := item // capture for goroutine
        g.Go(func() error {
            select {
            case sem <- struct{}{}:
                defer func() { <-sem }()
            case <-ctx.Done():
                return ctx.Err()
            }
            return process(ctx, item)
        })
    }
    return g.Wait()
}
```

### Pipeline pattern
```go
func generate(ctx context.Context, nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            select {
            case out <- n:
            case <-ctx.Done():
                return
            }
        }
    }()
    return out
}

func square(ctx context.Context, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-ctx.Done():
                return
            }
        }
    }()
    return out
}
```

### Fan-out / Fan-in
```go
func merge(ctx context.Context, channels ...<-chan Result) <-chan Result {
    var wg sync.WaitGroup
    merged := make(chan Result, len(channels))

    output := func(c <-chan Result) {
        defer wg.Done()
        for r := range c {
            select {
            case merged <- r:
            case <-ctx.Done():
                return
            }
        }
    }

    wg.Add(len(channels))
    for _, c := range channels {
        go output(c)
    }

    go func() {
        wg.Wait()
        close(merged)
    }()
    return merged
}
```

## Interface Patterns

### Functional options
```go
type Server struct {
    addr    string
    timeout time.Duration
    maxConn int
}

type Option func(*Server)

func WithTimeout(d time.Duration) Option {
    return func(s *Server) { s.timeout = d }
}

func WithMaxConn(n int) Option {
    return func(s *Server) { s.maxConn = n }
}

func NewServer(addr string, opts ...Option) *Server {
    s := &Server{addr: addr, timeout: 30 * time.Second, maxConn: 100}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

### Repository pattern
```go
type UserRepository interface {
    FindByID(ctx context.Context, id int64) (*User, error)
    Save(ctx context.Context, u *User) error
    Delete(ctx context.Context, id int64) error
}

// Implementation in data layer
type postgresUserRepo struct{ db *sql.DB }

func (r *postgresUserRepo) FindByID(ctx context.Context, id int64) (*User, error) {
    // ...
}

// Test double
type fakeUserRepo struct{ users map[int64]*User }
```

## HTTP Middleware

```go
type Middleware func(http.Handler) http.Handler

func Chain(h http.Handler, middlewares ...Middleware) http.Handler {
    for i := len(middlewares) - 1; i >= 0; i-- {
        h = middlewares[i](h)
    }
    return h
}

func Logger(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)
        slog.Info("request", "method", r.Method, "path", r.URL.Path,
            "duration", time.Since(start))
    })
}

// Graceful shutdown
srv := &http.Server{Addr: ":8080", Handler: Chain(mux, Logger, Auth)}

go func() {
    if err := srv.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
        log.Fatal(err)
    }
}()

<-sigChan
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
srv.Shutdown(ctx)
```

## CLI Structure (cobra)

```go
// cmd/root.go
var rootCmd = &cobra.Command{
    Use:   "mytool",
    Short: "My research tool",
    PersistentPreRunE: func(cmd *cobra.Command, args []string) error {
        return initConfig()
    },
}

func Execute() error {
    return rootCmd.Execute()
}

// cmd/process.go — subcommand
func init() {
    processCmd := &cobra.Command{
        Use:   "process [file]",
        Short: "Process input file",
        Args:  cobra.ExactArgs(1),
        RunE:  runProcess,
    }
    processCmd.Flags().StringP("output", "o", "", "output directory")
    processCmd.MarkFlagRequired("output")
    rootCmd.AddCommand(processCmd)
}
```

## Structured Logging (slog, Go 1.21+)

```go
logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelInfo,
}))
slog.SetDefault(logger)

// Usage
slog.Info("processing", "file", path, "size", size)
slog.Error("failed", "err", err, "file", path)

// With context
ctx = context.WithValue(ctx, loggerKey, logger.With("request_id", id))
```

## Performance: sync.Pool

```go
var bufPool = sync.Pool{
    New: func() any { return new(bytes.Buffer) },
}

func process(data []byte) string {
    buf := bufPool.Get().(*bytes.Buffer)
    defer func() {
        buf.Reset()
        bufPool.Put(buf)
    }()
    // use buf...
    return buf.String()
}
```
