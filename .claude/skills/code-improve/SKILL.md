---
name: code-improve
description: Analyze source code for improvements across security, performance, refactoring, code smells, and best practices. Supports Go, TypeScript, and Python with language-specific checks.
disable-model-invocation: true
---

This skill performs multi-pass source code analysis to find actionable improvements. It uses category-specific expert analysis via sub agents and produces a structured report with severity levels and concrete fix suggestions.

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

## Phase 2: File Discovery

1. Normalize the user's target into a glob pattern:
   - **Single file** (`internal/auth/handler.go`) → use as-is
   - **Directory** (`internal/auth` or `internal/auth/`) → expand recursively to `{target}/**/*.{go,ts,tsx,py}` (include all supported extensions, not just one language — a directory may be polyglot)
   - **Glob pattern** (`src/**/*.go`, `internal/**/*.{ts,tsx}`) → use as-is
   - **Repo root** (`.`) → warn the user about cost (5 parallel sub agents × many files) and confirm before proceeding
2. Use `Glob` with the normalized pattern to collect the list of target file paths.
3. Exclude common noise paths unless the user explicitly included them: `node_modules/`, `vendor/`, `dist/`, `build/`, `.git/`, `*_test.go` / `*.test.ts` / `test_*.py` (tests are usually analyzed separately, but include them if the user asked).
4. Detect the primary language(s) from file extensions of the collected files:
   - `.go` → Go
   - `.ts`, `.tsx` → TypeScript
   - `.py` → Python
   - Other → apply general analysis only
5. Collect the final list of file paths. **Do not read file contents** — sub agents will read them independently.
6. If available, note the project's linter configuration path (`.golangci.yml`, `.eslintrc.*`, `pyproject.toml`, `ruff.toml`) to pass to sub agents.

## Phase 3: Multi-Pass Analysis via Sub Agents

For each selected category, launch an `Agent` (sub agent, `subagent_type: "general-purpose"`) with:
- The category-specific persona and checklist (from below)
- The list of target file paths
- The detected language(s)
- The linter configuration path (if found)

**Launch all sub agents in a single message (multiple `Agent` tool calls in one response) so they run in parallel.** Sequential invocation defeats the purpose of this design.

Each sub agent independently:
1. For each detected target language, reads `checks/{lang}.md` (e.g., `checks/go.md`) and treats the `## {Category}` section as **illustrative examples** of what to look for — not an exhaustive list. Combine with the generic checks below and apply the same principles to unlisted but equivalent issues.
2. Reads `checks/patterns.md` and, when target code exhibits any pattern's **Trigger** (outbound HTTP client, DB query, cache, queue, background job, file I/O, logging, rate limiting, auth/session, config/feature flags), applies that pattern's **Checklist**. Pattern findings are language-agnostic and get classified under the most relevant category.
3. Reads the target files using `Read`
4. If a linter configuration exists, reads it to avoid duplicate findings
5. Performs analysis for its assigned category only
6. Returns findings as structured JSON

### Cross-cutting: Deprecated / Legacy Usage

Across all categories, sub agents should flag any code using APIs, syntax, or patterns known to be deprecated or legacy in the target language ecosystem. Rely on LLM knowledge — do not enumerate specific APIs. Each language has a deprecation convention that serves as a signal:

- **Go**: `Deprecated:` marker in godoc (`staticcheck` SA1019 territory)
- **Python**: `DeprecationWarning` / `PendingDeprecationWarning`; APIs removed or scheduled for removal in recent CPython releases
- **TypeScript / JavaScript**: `@deprecated` JSDoc annotation; legacy language features

When uncertain about the deprecation status of a specific API, note the uncertainty rather than reporting it as definitive (see `Avoid false positives` principle). Deprecated usage findings are classified under **Refactoring** unless they constitute a security or performance issue.

### Sub Agent Output Format

Each sub agent must return a JSON array:

```json
[
  {
    "id": "S1",
    "category": "Security",
    "title": "Brief title",
    "file": "path/to/file.go",
    "line": 42,
    "severity": "Critical|High|Medium|Low",
    "current_code": "// problematic code snippet",
    "issue": "Clear explanation of the problem and its impact",
    "suggested_fix": "// improved code snippet"
  }
]
```

### Category: Security (Persona: Senior Security Engineer)

Analyze for:
- **Injection vulnerabilities**: SQL injection, command injection, XSS, path traversal
- **Secrets exposure**: hardcoded credentials, API keys, tokens, passwords
- **Cryptographic issues**: weak hashing (MD5/SHA1 for security), insecure random, broken crypto
- **Unsafe deserialization**: untrusted data unmarshaling without validation
- **Missing input validation**: system boundary inputs not validated
- **Concurrency safety**: race conditions, unprotected shared state, goroutine leaks (Go)
- **Auth/AuthZ gaps**: missing authentication checks, privilege escalation paths

For language-specific security checks, see `checks/{lang}.md` → `## Security` section.

### Category: Performance (Persona: Performance Engineer)

Analyze for:
- **Algorithmic inefficiency**: O(n^2) or worse where O(n log n) or O(n) is possible
- **N+1 queries**: database calls in loops
- **Unnecessary allocations**: allocations in hot loops, string concatenation in loops
- **Missing caching**: repeated expensive computations with same inputs
- **Unbounded growth**: maps/slices/channels that grow without limits
- **Blocking in hot paths**: I/O or locks in performance-critical code
- **Unnecessary copies**: large struct copies where pointers would suffice

For language-specific performance checks, see `checks/{lang}.md` → `## Performance` section.

### Category: Refactoring (Persona: Software Architect)

Analyze for:
- **God functions/methods**: functions doing too many things (>40 lines, multiple responsibilities)
- **Deep nesting**: >3 levels of indentation (consider early returns, guard clauses)
- **Duplicated logic**: similar code blocks that should be abstracted
- **Inappropriate coupling**: tight coupling between modules that should be independent
- **Dead code**: unused functions, unreachable branches, commented-out code
- **Inconsistent error handling**: mixed patterns for error handling within the same module
- **Long parameter lists**: >4 parameters (consider parameter objects or builder pattern)
- **Primitive obsession**: using primitive types where domain types would be clearer

For language-specific refactoring checks, see `checks/{lang}.md` → `## Refactoring` section.

### Category: Code Smells (Persona: Code Quality Specialist)

Analyze for:
- **Feature envy**: methods that use another object's data more than their own
- **Shotgun surgery**: one conceptual change requires edits across many files
- **Magic numbers/strings**: unnamed constants scattered through code
- **Overly complex conditionals**: boolean expressions that need extraction or truth tables
- **Data clumps**: groups of variables that always appear together (should be a struct/class)
- **Middle man**: classes/functions that only delegate without adding value
- **Speculative generality**: abstractions for scenarios that don't exist yet
- **Temporal coupling**: operations that must happen in a specific order but nothing enforces it

For language-specific code smells, see `checks/{lang}.md` → `## Code Smells` section.

### Category: Best Practices (Persona: Language Expert)

Best practices are almost entirely language-idiomatic. Sub agents must read `checks/{lang}.md` → `## Best Practices` section for each detected language and treat it as the complete checklist for this category.

## Phase 4: Consolidation

Collect the JSON results from all sub agents, then:

1. **Deduplicate**: Remove findings that overlap across categories (keep in the most relevant category)
2. **Validate severity** assignments:
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

- **Judge from general principles** — This skill reasons from general principles, not from enumerated rule databases. The items in `checks/{lang}.md` and `checks/patterns.md` are **illustrative examples**, not exhaustive checklists. Sub agents must recognize equivalent issues through the underlying principle (e.g., "avoid unbounded resource consumption", "prefer modern idioms over legacy APIs", "validate at trust boundaries"), even when the specific API, library, or syntax is not explicitly listed. Use LLM knowledge of the language ecosystem to identify issues that rhyme with the listed examples. For exhaustive rule enforcement, defer to linters (`staticcheck`, `ruff`, `eslint`, etc.) — this skill's value is reasoning, not rule matching.
- **Never analyze from descriptions alone** — always read the actual code first.
- **Be specific** — every finding must include the exact file path and line reference.
- **Be actionable** — every finding must include a concrete code suggestion, not just a problem description.
- **Avoid false positives** — if unsure whether something is an issue, note the uncertainty rather than reporting it as definitive.
- **Respect existing patterns** — if the codebase consistently uses a pattern, don't flag it as an issue unless it's a genuine problem (security, correctness).
- **Consider context** — a pattern that's fine in a CLI tool may be problematic in a web server. Adjust severity based on the application context.
- **Don't flag what linters catch** — if the project has a linter configured, focus on issues beyond what the linter already enforces.
- **Suggest, don't demand** — present findings as recommendations for human review, not automatic fixes.
