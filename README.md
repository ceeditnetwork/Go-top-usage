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

## 11. Bonus: Testing

Built into the toolchain — no framework needed. Table-driven tests are the convention.

```go
func TestAdd(t *testing.T) {
    cases := []struct {
        a, b, want int
    }{
        {1, 2, 3},
        {0, 0, 0},
    }
    for _, c := range cases {
        if got := Add(c.a, c.b); got != c.want {
            t.Errorf("Add(%d,%d)=%d want %d", c.a, c.b, got, c.want)
        }
    }
}
```

Run with `go test ./...`. Add `-race` to catch data races and `-bench .` for benchmarks.

---

## Repo Layout

```
go-in-10-topics/
├── README.md
├── 01-goroutines/main.go
├── 02-structs/main.go
├── 03-interfaces/main.go
├── 04-errors/main.go
├── 05-slices-maps/main.go
├── 06-defer-panic-recover/main.go
├── 07-net-http/main.go
├── 08-json/main.go
├── 09-generics/main.go
├── 10-context/main.go
└── 11-testing/
    ├── add.go
    └── add_test.go
```

Each folder is runnable on its own with `go run ./NN-topic`.
