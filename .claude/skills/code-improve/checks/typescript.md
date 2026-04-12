# TypeScript-specific Checks

Items below are **illustrative examples grouped by principle**, not an exhaustive checklist. Sub agents must apply each principle broadly — if the code exhibits an issue that rhymes with a listed example but is not explicitly mentioned, report it. For exhaustive rule enforcement, defer to linters (`eslint`, `tsc --strict`).

## Security

**Never execute or inject untrusted input** — e.g.:
- `eval()` / `Function()` constructor on user input
- Passing strings to `setTimeout` / `setInterval` (acts like `eval`)
- `innerHTML` / `dangerouslySetInnerHTML` with unsanitized data
- `document.write` with user-controlled content

**Validate at trust boundaries** — e.g.:
- Unvalidated URL construction → open redirect, SSRF
- Prototype pollution (unsafe object merging on user input)
- `JSON.parse` without schema validation on untrusted input
- Missing CSRF protection on state-changing endpoints

**Protect secrets** — e.g.:
- Hardcoded secrets / API keys in frontend bundles

## Performance

**Don't block the event loop** — e.g.:
- Synchronous operations blocking the main thread
- Large JSON payloads parsed synchronously
- CPU-intensive work that should use Web Workers

**Parallelize independent async work** — e.g.:
- Sequential `await` where `Promise.all` / `Promise.allSettled` would work
- Missing debounce / throttle on frequent event handlers

**Optimize bundle and render cost** — e.g.:
- Large bundle imports where tree-shaking would work
- Unnecessary re-renders (missing `memo`, unstable deps in `useEffect`)
- Missing virtualization for long lists
- Missing code splitting / lazy loading at route boundaries

## Refactoring

**Leverage the type system** — e.g.:
- `any` type usage without justification
- Type assertions (`as X`) instead of proper type guards
- Missing discriminated unions where type narrowing would help
- `enum` where string literal union would work

**Use modern syntax** — e.g.:
- `var` usage (use `const` / `let`)
- `==` / `!=` instead of `===` / `!==`
- String concatenation with `+` instead of template literals
- Callback chains over `async` / `await`

**Keep components and modules focused** — e.g.:
- God components (>200 lines)
- Barrel file bloat (`index.ts` re-exporting too much)
- Long parameter lists (use options objects)

## Code Smells

- Magic strings / numbers in conditionals
- Mutating props / state directly
- `useEffect` dependencies with unstable object references
- Conditional hooks (violates rules of hooks)
- Commented-out JSX / code left in source
- `console.log` left in production code
- Boolean variables without `is` / `has` / `should` / `can` prefix
- Functions with hidden side effects
- `switch` without `default` or `never` exhaustiveness check

## Best Practices

**Strict type safety** — `strict: true` in tsconfig; no `any` without justification; `readonly` / `as const` for immutability; discriminated unions over type assertions; `satisfies` operator; type-only imports.

**Equality and syntax** — `===` / `!==`; `const` / `let` (never `var`); curly braces on all blocks; template literals; literal initializers.

**Error handling** — `throw new Error(...)` (never plain objects/strings); `new Error('msg', { cause: err })` for chaining; try/catch around fallible `await`.

**Async** — `async` / `await` over raw Promise chains; `Promise.all` for parallel work; `AbortController` for cancellation.

**Functions** — pure where feasible; centralize side effects; >3 params → options object; return early to avoid nesting.

**Naming** — `is` / `has` / `should` / `can` prefix for booleans; meaningful names; English identifiers.

**Module organization** — imports ordered (external → internal → relative); no global variables; export only what's needed.
