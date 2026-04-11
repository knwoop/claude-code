---
name: code-improve
description: Analyze source code for improvements across security, performance, refactoring, code smells, and best practices. Supports Go, TypeScript, and Python with language-specific checks.
disable-model-invocation: true
---

This skill performs multi-pass source code analysis to find actionable improvements. It uses category-specific expert analysis and produces a structured report with severity levels and concrete fix suggestions.

## Phase 1: Scope Clarification

Use `AskUserQuestion` to ask the user:

1. **Target**: Which files or directories to analyze? (Accept glob patterns like `src/**/*.go`)
2. **Categories**: Which improvement categories to check?
   - All (default)
   - Security
   - Performance
   - Refactoring
   - Code Smells
   - Best Practices
3. **Depth**: Quick scan (top issues only) or deep analysis (comprehensive)?

If `$ARGUMENTS` is provided, treat it as the target path and default to all categories with deep analysis. Still confirm with the user before proceeding.

## Phase 2: Code Reading

Before any analysis:

1. Use `Glob` to discover all target files matching the user's scope.
2. Detect the primary language(s) from file extensions:
   - `.go` → Go
   - `.ts`, `.tsx` → TypeScript
   - `.py` → Python
   - Other → apply general analysis only
3. Use `Read` to read each target file. For large files (>500 lines), read in segments.
4. Note the project structure, module boundaries, and existing patterns.
5. If available, read the project's linter configuration (`.golangci.yml`, `.eslintrc.*`, `pyproject.toml`, `ruff.toml`) to understand existing rules and avoid duplicate findings.

## Phase 3: Multi-Pass Analysis

Run a separate analysis pass for each selected category. For each pass, adopt the specified expert persona and focus exclusively on that category.

### Pass 1: Security (Persona: Senior Security Engineer)

Analyze for:
- **Injection vulnerabilities**: SQL injection, command injection, XSS, path traversal
- **Secrets exposure**: hardcoded credentials, API keys, tokens, passwords
- **Cryptographic issues**: weak hashing (MD5/SHA1 for security), insecure random, broken crypto
- **Unsafe deserialization**: untrusted data unmarshaling without validation
- **Missing input validation**: system boundary inputs not validated
- **Concurrency safety**: race conditions, unprotected shared state, goroutine leaks (Go)
- **Auth/AuthZ gaps**: missing authentication checks, privilege escalation paths

Language-specific security checks:
- **Go**: `unsafe` package usage, unchecked `os/exec`, SQL string concatenation, missing `context.Context` for cancellation
- **TypeScript**: `eval()`, `innerHTML`, `dangerouslySetInnerHTML`, unvalidated URL construction, prototype pollution
- **Python**: `pickle.loads()` on untrusted data, `subprocess.shell=True`, `eval()`/`exec()`, YAML `load()` vs `safe_load()`

### Pass 2: Performance (Persona: Performance Engineer)

Analyze for:
- **Algorithmic inefficiency**: O(n^2) or worse where O(n log n) or O(n) is possible
- **N+1 queries**: database calls in loops
- **Unnecessary allocations**: allocations in hot loops, string concatenation in loops
- **Missing caching**: repeated expensive computations with same inputs
- **Unbounded growth**: maps/slices/channels that grow without limits
- **Blocking in hot paths**: I/O or locks in performance-critical code
- **Unnecessary copies**: large struct copies where pointers would suffice

Language-specific performance checks:
- **Go**: unnecessary `append` pre-allocation missing, `string([]byte)` conversions in loops, sync.Pool opportunities, excessive goroutine creation, `defer` in tight loops
- **TypeScript**: synchronous operations blocking event loop, missing `Promise.all` for parallel async, large bundle imports where tree-shaking is possible, unnecessary re-renders (React)
- **Python**: list comprehension vs generator for large datasets, missing `__slots__`, GIL-bound CPU work that should use multiprocessing

### Pass 3: Refactoring (Persona: Software Architect)

Analyze for:
- **God functions/methods**: functions doing too many things (>40 lines, multiple responsibilities)
- **Deep nesting**: >3 levels of indentation (consider early returns, guard clauses)
- **Duplicated logic**: similar code blocks that should be abstracted
- **Inappropriate coupling**: tight coupling between modules that should be independent
- **Dead code**: unused functions, unreachable branches, commented-out code
- **Inconsistent error handling**: mixed patterns for error handling within the same module
- **Long parameter lists**: >4 parameters (consider parameter objects or builder pattern)
- **Primitive obsession**: using primitive types where domain types would be clearer

Language-specific refactoring checks:
- **Go**: error wrapping with `%w` vs `%v`, table-driven tests opportunities, interface segregation (large interfaces that should be split), exported names that could be unexported
- **TypeScript**: `any` type usage, missing discriminated unions, callback hell vs async/await, barrel file bloat
- **Python**: missing dataclasses/NamedTuple for data containers, bare `except:` clauses, mutable default arguments

### Pass 4: Code Smells (Persona: Code Quality Specialist)

Analyze for:
- **Feature envy**: methods that use another object's data more than their own
- **Shotgun surgery**: one conceptual change requires edits across many files
- **Magic numbers/strings**: unnamed constants scattered through code
- **Overly complex conditionals**: boolean expressions that need extraction or truth tables
- **Data clumps**: groups of variables that always appear together (should be a struct/class)
- **Middle man**: classes/functions that only delegate without adding value
- **Speculative generality**: abstractions for scenarios that don't exist yet
- **Temporal coupling**: operations that must happen in a specific order but nothing enforces it

### Pass 5: Best Practices (Persona: Language Expert)

**Go best practices:**
- Proper error wrapping with `fmt.Errorf("...: %w", err)`
- Context propagation through call chains
- Goroutine lifecycle management (ensure goroutines can be stopped)
- Interface design: accept interfaces, return concrete types
- Meaningful variable names (avoid single-letter except `i`, `j`, `k` in loops, `err`, `ctx`)
- Proper use of `sync` primitives
- Table-driven tests with clear test case names

**TypeScript best practices:**
- Strict typing (no `any` without justification)
- Null/undefined safety (optional chaining, nullish coalescing)
- Proper async/await error handling (try/catch around await)
- Immutability where appropriate (`readonly`, `as const`)
- Discriminated unions over type assertions
- Proper module organization and exports

**Python best practices:**
- Type hints for function signatures and complex variables
- Context managers for resource management (`with` statements)
- Specific exception types (never bare `except:`)
- Pathlib over os.path for file operations
- f-strings over `.format()` or `%` formatting
- Proper `__init__.py` and module structure
- Docstrings for public API functions

## Phase 4: Consolidation

After all passes complete:

1. **Deduplicate**: Remove findings that overlap across categories (keep in the most relevant category)
2. **Assign severity**:
   - **Critical**: Security vulnerabilities that could be exploited, data loss risks
   - **High**: Significant performance issues, major design flaws, potential bugs
   - **Medium**: Code smells that impact maintainability, minor performance issues
   - **Low**: Style issues, minor best practice deviations, documentation gaps
3. **Sort**: Order findings by severity (Critical → High → Medium → Low), then by file
4. **Count**: Tally findings per category and severity

## Phase 5: Report

Present findings in this format:

```markdown
# Code Improvement Report

## Summary
- **Files analyzed**: N
- **Language(s)**: Go / TypeScript / Python
- **Findings**: N total (N Critical, N High, N Medium, N Low)

## Critical Findings

### [C1] Category: Brief title
- **File**: `path/to/file.go:42`
- **Severity**: Critical
- **Current code**:
  ```go
  // problematic code snippet
  ```
- **Issue**: Clear explanation of the problem and its impact
- **Suggested fix**:
  ```go
  // improved code snippet
  ```

## High Findings
...

## Medium Findings
...

## Low Findings
...

## Next Steps
- Prioritized list of recommended actions
- Suggestion to run linter after applying fixes (if applicable)
```

## Principles

- **Never analyze from descriptions alone** — always read the actual code first.
- **Be specific** — every finding must include the exact file path and line reference.
- **Be actionable** — every finding must include a concrete code suggestion, not just a problem description.
- **Avoid false positives** — if unsure whether something is an issue, note the uncertainty rather than reporting it as definitive.
- **Respect existing patterns** — if the codebase consistently uses a pattern, don't flag it as an issue unless it's a genuine problem (security, correctness).
- **Consider context** — a pattern that's fine in a CLI tool may be problematic in a web server. Adjust severity based on the application context.
- **Don't flag what linters catch** — if the project has a linter configured, focus on issues beyond what the linter already enforces.
- **Suggest, don't demand** — present findings as recommendations for human review, not automatic fixes.
