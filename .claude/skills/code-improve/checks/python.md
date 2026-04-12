# Python-specific Checks

Items below are **illustrative examples grouped by principle**, not an exhaustive checklist. Sub agents must apply each principle broadly — if the code exhibits an issue that rhymes with a listed example but is not explicitly mentioned, report it. For exhaustive rule enforcement, defer to linters (`ruff`, `flake8`, `mypy`).

## Security

**Never execute or deserialize untrusted input** — e.g.:
- `pickle.loads()` / `pickle.load()` on untrusted data
- `eval()` / `exec()` on user input
- `subprocess(..., shell=True)` with user input
- YAML `load()` without `safe_load()`
- `xml.etree` / `lxml` without `defusedxml`

**Validate at trust boundaries** — e.g.:
- SQL string concatenation / `.format()` in queries (use parameterized queries)
- `Jinja2` autoescape disabled for HTML
- `requests` with `verify=False`

**Protect secrets and credentials** — e.g.:
- Hardcoded secrets / credentials in source
- Flask / Django debug mode in production
- Weak hashing (`hashlib.md5` / `sha1`) for security purposes

## Performance

**Choose the right abstraction for scale** — e.g.:
- List comprehension vs generator for large datasets (memory)
- Missing `__slots__` on frequently-instantiated data classes
- GIL-bound CPU work that should use `multiprocessing`
- String concatenation in loops (use `"".join(...)`)

**Avoid blocking in async contexts** — e.g.:
- Sync I/O in async handlers
- Missing `asyncio.gather` for parallel async
- Sequential awaits where parallelism is possible

**Watch for ORM pitfalls** — e.g.:
- N+1 queries (missing `select_related` / `prefetch_related`)
- Pandas `iterrows` where vectorized ops work

## Refactoring

- Missing `dataclass` / `NamedTuple` for data containers
- Bare `except:` clauses (catch specific exceptions)
- Mutable default arguments (`def f(x=[])`)
- Long function signatures without keyword-only arguments
- Classes used where a function / module would suffice
- `__init__.py` with heavy logic / side effects
- Circular imports

## Code Smells

- Magic numbers / strings without constants
- `global` keyword usage
- Overly complex list / dict comprehensions
- Deep nesting where early return / guard clauses work
- `print` debugging left in source
- Commented-out code blocks
- `lambda` assigned to variables (use `def`)

## Best Practices

**Type safety** — type hints for public function signatures and complex variables.

**Resource management** — context managers (`with` statements) for files, connections, locks.

**Error handling** — specific exception types, never bare `except:`.

**Modern idioms** — `pathlib.Path` over `os.path`; f-strings over `.format()`; `enum.Enum` / `StrEnum` over string constants; `dataclass(frozen=True)` / `NamedTuple` for immutable data.

**Module hygiene** — `logging` module over `print`; `__all__` for explicit exports; clean `__init__.py`.
