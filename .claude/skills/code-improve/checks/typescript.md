# TypeScript-specific Checks

TypeScript checklist for the `code-improve` skill. Sub agents should read the `## {Category}` section matching their assigned category and combine it with the generic checks in `SKILL.md`.

## Security
- `eval()` / `Function()` constructor on user input
- Passing strings to `setTimeout` / `setInterval` (acts like `eval`)
- `innerHTML` / `outerHTML` with unsanitized data
- `dangerouslySetInnerHTML` (React) with unsanitized data
- Unvalidated URL construction → open redirect, SSRF
- Prototype pollution (unsafe object merging, `Object.assign` on user input)
- `JSON.parse` without try/catch or schema validation on untrusted input
- Hardcoded secrets / API keys in frontend bundles
- Missing CSRF protection on state-changing endpoints
- Weak / missing Content-Security-Policy on rendered HTML
- `document.write` with user-controlled content

## Performance
- Synchronous operations blocking the event loop
- Missing `Promise.all` / `Promise.allSettled` for parallel async
- Sequential `await` where parallelism is possible
- Large bundle imports where tree-shaking would work (`import * as lib from 'lib'`)
- Unnecessary re-renders (React: missing `memo`, unstable deps in `useEffect` / `useMemo`)
- Missing virtualization for long lists
- Repeated work in render without `useMemo` / `useCallback`
- Large JSON payloads parsed synchronously
- Missing debounce / throttle on frequent event handlers
- CPU-intensive work on main thread (candidate for Web Workers)
- Missing code splitting / lazy loading at route or feature boundaries

## Refactoring

### Type System
- `any` type usage without justification
- Missing discriminated unions where type narrowing would help
- Long type intersection chains that should be named types
- Type assertions (`as X`) instead of proper type guards
- `unknown` cast directly to concrete type without validation
- `enum` where string literal union would work
- `namespace` where ES modules would work

### Syntax & Legacy
- `var` usage (use `const` / `let`)
- `==` / `!=` instead of `===` / `!==`
- Omitted curly braces in `if` / `for` / `while` (even one-liners)
- String concatenation with `+` instead of template literals
- `new Object()` / `new Array()` / `new String()` / `new Boolean()` — use literals `{}`, `[]`, `""`, `false`
- Classic `for` loops where `for-of` / array methods read better (unless measurably hot)

### Async
- Callback hell vs `async`/`await`
- Raw Promise chains (`.then().then()`) over `async`/`await`
- Missing try/catch around `await`

### Organization
- Barrel file bloat (`index.ts` re-exporting too much)
- God components (React components with >200 lines)
- Long parameter lists — use options objects

## Code Smells
- Magic strings / numbers in conditionals
- Deeply nested object types (should be split)
- Mutating props / state directly
- Mutating native prototypes (`Array.prototype.foo = ...`)
- `useEffect` dependencies with unstable object references
- Conditional hooks (violates rules of hooks)
- Commented-out JSX / code left in source
- `console.log` left in production code
- Boolean variables without `is` / `has` / `should` / `can` prefix
- `with` keyword, `void` operator, `eval` — "weird JS features"
- Functions with hidden side effects (writes to globals, mutates arguments)
- `switch` without `default` — especially without `never` exhaustiveness check
- `throw "string"` / `throw { message: ... }` — not an `Error` instance
- Technology-coupled naming (`isOverEighteen` → `isLegalDrinkingAge`)

## Best Practices

### tsconfig Strict Flags
- `strict: true` (umbrella flag)
- Individual flags covered by `strict`: `noImplicitAny`, `strictNullChecks`, `noImplicitThis`, `alwaysStrict`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`
- `noImplicitReturns`, `noUnusedLocals`, `noUnusedParameters`
- `noFallthroughCasesInSwitch`, `exactOptionalPropertyTypes`
- `forceConsistentCasingInFileNames`

### Type System
- Strict typing — no `any` without justification
- Null / undefined safety (`?.`, `??`)
- `readonly`, `as const`, `ReadonlyArray<T>` for immutability
- Discriminated unions over type assertions
- String literal unions for finite sets (`type Status = 'active' | 'inactive'`)
- Utility types: `Partial<T>`, `Readonly<T>`, `Required<T>`, `Pick<T, K>`, `Omit<T, K>`, `Record<K, V>`, `ReturnType<F>`, `Parameters<F>`, `Awaited<T>`
- `satisfies` operator to validate shape without widening
- Type-only imports (`import type`) for type-only dependencies
- Prefer `interface` for public / extensible APIs, `type` for unions and utility types
- Exhaustiveness checking with `never` in switch default

### Equality & Syntax
- `===` / `!==` over `==` / `!=`
- `const` / `let` — never `var`
- Always use curly braces for `if` / `for` / `while` bodies
- Template literals over string concatenation
- Literal initializers (`{}`, `[]`, `""`) over constructor forms

### Errors
- `throw new Error(...)` — never throw plain objects, strings, or numbers
- Custom error classes extending `Error` for typed handling (`errors.Is`/`As` equivalent via `instanceof`)
- Try/catch around every `await` that can fail (or `.catch` on the promise)
- Reject promises with `Error` instances, not plain values
- Preserve the cause with `new Error('msg', { cause: err })`

### Async
- `async`/`await` over raw Promise chains / callbacks
- `Promise.all` / `Promise.allSettled` for parallel independent operations
- Never pass strings to `setTimeout` / `setInterval`
- Cancel in-flight work with `AbortController` when appropriate

### Functions
- Pure functions where feasible (testability, memoization, parallelism)
- Centralize side effects — don't scatter them through pure logic
- Parameter defaults over explicit `undefined` checks
- Reduce parameter count (>3 → options object)
- No boolean flag parameters — split the function
- Return early; avoid deep nesting

### Naming
- `is` / `has` / `should` / `can` prefix for Booleans
- Meaningful names — no `x1`, `fe2`, `tmp`
- English identifiers (programming languages are in English)

### Module Organization
- Organize imports (external → internal → relative)
- Remove unused imports
- Avoid global variables and global state
- Never modify native prototypes
- `export` only what the outside needs

### Testing
- Write tests for every new feature / module
- Type-safe test fixtures (no `any` in tests)
