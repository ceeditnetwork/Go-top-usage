# Go in 10 Topics

A hands-on tour of the Go features you actually reach for every day. Each topic is a self-contained chapter with a runnable `main.go`.

## Table of Contents

1. [Goroutines & Channels](#1-goroutines--channels)
2. [Structs & Methods](#2-structs--methods)
3. [Interfaces](#3-interfaces)
4. [Error Handling](#4-error-handling)
5. [Slices & Maps](#5-slices--maps)
6. [defer / panic / recover](#6-defer--panic--recover)
7. [net/http](#7-nethttp)
8. [JSON & Struct Tags](#8-json--struct-tags)
9. [Generics](#9-generics)
10. [context.Context](#10-contextcontext)
11. [Bonus: Testing](#11-bonus-testing)

---

## 1. Goroutines & Channels

Go's signature feature: lightweight concurrency built into the language.

```go
ch := make(chan int)
go func() { ch <- 42 }() // launch concurrent work
fmt.Println(<-ch)        // receive
```

A goroutine is a function running concurrently; a channel is how goroutines communicate and synchronize. Prefer "share memory by communicating" over locks where you can.

## 2. Structs & Methods

Go's data + behavior model — no classes, just structs with methods attached.

```go
type User struct {
    Name string
    Age  int
}

func (u User) Greet() string { return "Hi " + u.Name }
```

Use a pointer receiver (`func (u *User)`) when the method needs to mutate the struct or the struct is large.

## 3. Interfaces

Satisfied implicitly — any type with the right methods qualifies, no `implements` keyword.

```go
type Stringer interface {
    String() string
}
// Any type with a String() string method satisfies Stringer automatically.
```

Keep interfaces small. The idiom is to accept interfaces and return concrete types.

## 4. Error Handling

Errors are ordinary values, returned explicitly — no exceptions.

```go
if err != nil {
    return fmt.Errorf("loading config: %w", err)
}
// Inspect wrapped errors later:
//   errors.Is(err, os.ErrNotExist)
//   var perr *os.PathError; errors.As(err, &perr)
```

Wrap with `%w` to preserve the chain; unwrap with `errors.Is` / `errors.As`.

## 5. Slices & Maps

The two workhorse collections.

```go
s := []int{1, 2}
s = append(s, 3) // append may reallocate

m := map[string]int{"a": 1}
v, ok := m["a"] // comma-ok tells you if the key exists
```

A slice is a view over a backing array (pointer, length, capacity). Maps are unordered and not safe for concurrent writes.

## 6. defer / panic / recover

Cleanup that runs when the function returns, plus controlled recovery.

```go
f, _ := os.Open("x")
defer f.Close() // runs on return, even on panic

defer func() {
    if r := recover(); r != nil {
        // handle the panic instead of crashing
    }
}()
```

Deferred calls run last-in-first-out. Reserve `panic`/`recover` for truly exceptional cases, not normal control flow.

## 7. net/http

Production-grade servers and clients straight from the standard library.

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "hello")
})
http.ListenAndServe(":8080", nil)
```

For real services, build your own `*http.Server` with timeouts and a custom `*http.ServeMux`.

## 8. JSON & Struct Tags

Encoding and decoding JSON, ubiquitous in APIs.

```go
type Cfg struct {
    Port int `json:"port"`
}

var cfg Cfg
json.Unmarshal(data, &cfg)
out, _ := json.Marshal(cfg)
```

Struct tags control field names and options (`json:"name,omitempty"`). Exported (capitalized) fields only are (de)serialized.

## 9. Generics

Type parameters for reusable code, available since Go 1.18.

```go
func Map[T, U any](in []T, f func(T) U) []U {
    out := make([]U, len(in))
    for i, v := range in {
        out[i] = f(v)
    }
    return out
}
```

Use constraints (`any`, `comparable`, or your own interface) to bound type parameters. Reach for generics when you'd otherwise duplicate logic across types, not by default.

## 10. context.Context

Cancellation, deadlines, and request-scoped values threaded through call chains.

```go
ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
```

Pass `ctx` as the first parameter, propagate it down, and always call `cancel` to release resources.

## 11. sync: Mutex, WaitGroup, Once

When channels are overkill, the sync package gives you direct coordination primitives.
```
var (
    mu      sync.Mutex
    wg      sync.WaitGroup
    once    sync.Once
    counter int
)

for i := 0; i < 5; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        mu.Lock()
        counter++ // guarded against concurrent writes
        mu.Unlock()
    }()
}
wg.Wait() // block until all goroutines finish

once.Do(func() { /* runs exactly once, even across goroutines */ })
```

Use WaitGroup to wait for a set of goroutines, Mutex to protect shared state, and Once for lazy one-time init. Run with -race to confirm your locking is correct.

## 12. io.Reader & io.Writer

Two tiny interfaces that almost the entire standard library speaks. Anything that produces or consumes a stream of bytes implements one of them, so files, network connections, buffers, and HTTP bodies all compose interchangeably.

```
type Reader interface{ Read(p []byte) (n int, err error) }
type Writer interface{ Write(p []byte) (n int, err error) }

// Stream from any source to any destination without knowing the concrete types:
n, err := io.Copy(os.Stdout, strings.NewReader("stream me"))
```

Because *os.File, *bytes.Buffer, net.Conn, and http.Request.Body all satisfy these, helpers like io.Copy, bufio.Scanner, and json.NewDecoder(r) work on all of them. Accept io.Reader/io.Writer in your own APIs and they'll plug into everything.
