# Go-specific Checks

Exhaustive Go checklist for the `code-improve` skill. Sub agents should read the `## {Category}` section matching their assigned category and combine it with the generic checks in `SKILL.md`.

## Security

### Injection / Path / URL
- SQL string concatenation (use `database/sql` placeholders)
- `os/exec.Command` with shell interpretation (`sh -c`) or unsanitized user input
- `filepath.Join` without `filepath.Clean` → path traversal
- `http.ServeFile` / `http.Dir` with user-controlled paths
- `html/template` vs `text/template` misuse → XSS
- `text/template` used for HTML output
- Unvalidated outbound URLs in `http.Get` / `http.Client` → SSRF

### Crypto / Auth
- `math/rand` instead of `crypto/rand` for tokens, nonces, session IDs
- `crypto/tls` with `InsecureSkipVerify: true`
- JWT parsing without algorithm verification (`alg: none` attack)
- MD5 / SHA1 for security (password hashing, integrity)
- Password hashing with anything other than bcrypt / scrypt / argon2
- Hardcoded keys, API tokens, credentials

### DoS / Resource Exhaustion
- Unbounded `io.ReadAll` on request body → memory exhaustion
- Missing `http.MaxBytesReader` for request size limits
- `http.Server` / `http.Client` without timeouts (slowloris, SSRF amplification)
- Unbounded goroutine creation per request
- Goroutine leaks from missing `context.Context` propagation
- Regex with catastrophic backtracking on user input (ReDoS)
- Unbounded channel buffering

### Deserialization
- `encoding/xml` XXE / billion laughs attacks
- `encoding/gob` decoding untrusted data
- `encoding/json` unbounded nesting depth
- YAML deserialization without type restrictions

### File / Filesystem
- `os.OpenFile` with world-writable modes (`0666`, `0777`)
- TOCTOU races (`os.Stat` then `os.Open`)
- `ioutil.TempFile` with predictable paths (use `os.CreateTemp`)
- `os.Create` / `os.Remove` on user-controlled paths without validation

### Information Leaks
- Error messages exposing internal paths, SQL statements, stack traces
- Panic not recovered at HTTP boundary → stack trace leak
- `net/http/pprof` exposed in production
- Debug endpoints mounted on production handlers

### Other
- `unsafe` package usage
- Type assertion without comma-ok idiom → panic DoS
- Outdated dependencies (`govulncheck` violations)
- `reflect` exposing unexported fields

## Performance

### Allocation
- `append` without pre-allocated capacity (`make([]T, 0, n)`)
- `string([]byte)` / `[]byte(string)` conversions in hot loops
- String concatenation in loops (use `strings.Builder`)
- `fmt.Sprintf` where simple concatenation / `strconv` works
- Map without size hint (`make(map[K]V, n)`)
- `json.Marshal` / `json.Unmarshal` in hot paths (consider code-gen like easyjson)
- Unnecessary interface boxing in hot paths

### Concurrency
- Goroutine-per-request without a worker pool
- `sync.Mutex` where `sync.RWMutex` or `sync/atomic` would suffice
- `sync.Pool` opportunities for frequently allocated objects
- Unbuffered channels in high-throughput paths

### Hot Path Waste
- `defer` inside tight loops (overhead accumulates)
- Regex compilation inside loops (hoist to package-level `var re = regexp.MustCompile(...)`)
- `time.Now()` / `log.Print` in hot loops
- `reflect` in hot paths
- Large struct copies where pointer would suffice
- `fmt.Println` / `log` calls on hot paths

### Algorithmic (Go-specific nuances)
- Linear scan of large slice where sorted + `sort.Search` would work
- Redundant recomputation without caching
- Missing `context.Context` cancellation → doing work that will be discarded
- Map iteration where slice would suffice (order-dependent logic)

## Refactoring

### Error Handling
- `fmt.Errorf` with `%v` instead of `%w` for wrapping
- Sentinel error compared with `==` instead of `errors.Is`
- Type assertion instead of `errors.As`
- Mixed error handling patterns within the same package
- `log.Fatal` outside `main` / `init`
- Silently ignored errors (`_, _ = `, blank receivers)

### Interface / Types
- Large interfaces (>5 methods) — violate interface segregation
- Accept concrete types, return interfaces (reverse of idiom)
- `interface{}` / `any` where generics fit
- Missing domain types (`type UserID string` over `string`)
- Struct with >10 fields (consider decomposition)

### API Design
- Functions with >4 parameters (use options struct / functional options)
- Boolean flag parameters (use typed enums)
- Multiple return values >3 (use named struct)
- Missing `context.Context` as first parameter
- `context.Context` stored in struct fields
- `init()` with global state or hidden side effects
- Exported symbols that could be unexported

### Testing
- Non-table-driven tests where parametrization fits
- Missing `t.Helper()` in test helpers
- `defer` instead of `t.Cleanup`
- Test setup without subtests (`t.Run`)

### Package Organization
- Package-level mutable state
- Circular package dependencies
- Deep package hierarchies
- Utility packages (`utils`, `common`, `helpers`)

## Code Smells

- Magic numbers in switch / if chains
- `panic` where `error` return is appropriate
- Global logger instances threaded through package globals
- Commented-out code blocks left in source
- Stringly-typed APIs (string constants without a named type)
- Boolean flag parameters
- Premature goroutines (concurrency without clear need)
- `defer` after error-prone operation (never runs on early error return)
- Unused exported API
- `fmt.Println` / `println` debug prints in production code
- `TODO` / `FIXME` without issue reference
- Inconsistent sentinel vs typed errors within the same package
- `init()` abuse creating hidden ordering dependencies

## Best Practices

### Error Handling
- Wrap with `fmt.Errorf("context: %w", err)`
- Inspect with `errors.Is` / `errors.As`
- Sentinel errors at package level
- Return errors; panic only for programmer bugs

### Context
- `context.Context` as first parameter of any I/O / RPC function
- Propagate through call chains
- Never store in struct fields
- Check `ctx.Err()` in long-running loops

### Concurrency
- Every goroutine has a clear cancellation path
- Document goroutine-safety on exported APIs
- Use `sync/errgroup` for parallel operations with error aggregation
- Channels for signaling, mutexes for state

### Interfaces
- Accept interfaces, return concrete types
- Small interfaces (1-3 methods)
- Define at point of use, not point of implementation

### Naming
- Avoid stutter (`http.HTTPServer` → `http.Server`)
- Conventional short names: `ctx`, `err`, `i`/`j`/`k` for loop indices
- Short names in small scopes, descriptive in large scopes

### Testing
- Table-driven tests with `t.Run` subtests
- `t.Helper()` in helpers
- `t.Cleanup` over `defer`
- `cmp.Diff` over `reflect.DeepEqual`
- `testing.TB` for helpers shared between `testing.T` and `testing.B`

### Project Structure
- `internal/` for non-exported packages
- `cmd/` for binaries
- Zero-value useful structs where possible
- Package names: single word, lowercase, noun
