# Common Go Pitfalls

## Goroutine Leaks

```go
// WRONG: goroutine blocks forever if ctx is not cancelled
func startWorker(ctx context.Context) {
    go func() {
        for {
            select {
            case msg := <-messages:  // messages might never close
                process(msg)
            }
            // no ctx.Done case — will leak if messages stalls
        }
    }()
}

// RIGHT: always include ctx.Done
go func() {
    for {
        select {
        case msg, ok := <-messages:
            if !ok { return }
            process(msg)
        case <-ctx.Done():
            return
        }
    }
}()
```

## Nil Map Panic

```go
// WRONG: nil map panics on write
var m map[string]int
m["key"] = 1  // panic: assignment to entry in nil map

// RIGHT
m := make(map[string]int)
// or
m := map[string]int{}
```

## Defer in Loop

```go
// WRONG: defer runs at function return, not loop iteration
for _, f := range files {
    fh, _ := os.Open(f)
    defer fh.Close()  // all closes happen at function end = resource leak
    process(fh)
}

// RIGHT: close in a nested function or close explicitly
for _, f := range files {
    func() {
        fh, _ := os.Open(f)
        defer fh.Close()  // runs when inner func returns
        process(fh)
    }()
}
```

## Loop Variable Capture (pre-Go 1.22)

```go
// WRONG (Go < 1.22): all goroutines share the same variable
for _, v := range values {
    go func() {
        fmt.Println(v)  // all print the last value
    }()
}

// RIGHT: capture explicitly (or use Go 1.22+ which fixes this)
for _, v := range values {
    v := v  // new variable per iteration
    go func() {
        fmt.Println(v)
    }()
}
```

## Interface Nil Trap

```go
// WRONG: interface holding a typed nil is not nil
func getError() error {
    var err *MyError = nil
    return err  // returns non-nil interface!
}

if err := getError(); err != nil {  // true! surprise
    log.Fatal(err)
}

// RIGHT: return untyped nil
func getError() error {
    return nil
}
```

## Mutex Copy

```go
// WRONG: copying a mutex (embedded or direct) breaks it
type Cache struct {
    mu sync.Mutex
    data map[string]string
}

func useCache(c Cache) { ... }  // copies the mutex — undefined behavior

// RIGHT: always use pointer receiver or pass pointer
func useCache(c *Cache) { ... }
```

## Slice Aliasing

```go
// WRONG: subslice shares underlying array
original := []int{1, 2, 3, 4, 5}
sub := original[1:3]
sub[0] = 99  // modifies original[1]!

// RIGHT: copy if you need independence
sub := make([]int, 2)
copy(sub, original[1:3])
```

## Error Shadowing

```go
// WRONG: err in inner scope shadows outer err
var result *Result
result, err := fetchData()  // declares new err in outer scope
if err != nil {
    // ...
}
// then in a block:
{
    data, err := processResult(result)  // new err, old err still in scope
    // ...
}
// outer err still refers to fetchData error — can cause logic bugs

// RIGHT: use consistent scoping or explicit assignment
var err error
result, err = fetchData()
```

## Context Misuse

```go
// WRONG: storing values in context with built-in types as keys
ctx = context.WithValue(ctx, "userID", id)  // collision-prone

// RIGHT: use unexported custom type as key
type contextKey string
const userIDKey contextKey = "userID"
ctx = context.WithValue(ctx, userIDKey, id)
userID := ctx.Value(userIDKey).(int)
```

## String Concatenation in Loop

```go
// WRONG: O(n²) allocations
var result string
for _, s := range strs {
    result += s
}

// RIGHT: O(n)
var sb strings.Builder
sb.Grow(totalLen)  // optional pre-allocation
for _, s := range strs {
    sb.WriteString(s)
}
result := sb.String()
```
