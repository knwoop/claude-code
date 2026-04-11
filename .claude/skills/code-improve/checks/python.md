# Python-specific Checks

Python checklist for the `code-improve` skill. Sub agents should read the `## {Category}` section matching their assigned category and combine it with the generic checks in `SKILL.md`.

## Security
- `pickle.loads()` / `pickle.load()` on untrusted data
- `subprocess(..., shell=True)` with user input
- `eval()` / `exec()` on user input
- YAML `load()` without `safe_load()`
- `xml.etree` / `lxml` without `defusedxml` → XXE / billion laughs
- Hardcoded secrets / credentials
- `requests` with `verify=False`
- Weak hashing (`hashlib.md5` / `sha1`) for security purposes
- `os.system` / `os.popen` with user input
- Flask / Django debug mode in production
- SQL string concatenation / `.format()` in queries (use parameterized queries)
- `Jinja2` autoescape disabled for HTML

## Performance
- List comprehension vs generator for large datasets (memory)
- Missing `__slots__` on frequently-instantiated data classes
- GIL-bound CPU work that should use `multiprocessing`
- Repeated global lookups in tight loops (cache as locals)
- String concatenation in loops (use `"".join(...)`)
- `copy.deepcopy` where shallow copy suffices
- N+1 queries in ORM usage (missing `select_related` / `prefetch_related`)
- Sync I/O in async handlers
- Missing `asyncio.gather` for parallel async
- Pandas `iterrows` / `itertuples` where vectorized ops work

## Refactoring
- Missing `dataclass` / `NamedTuple` for data containers
- Bare `except:` clauses
- Mutable default arguments (`def f(x=[])`)
- Long function signatures without keyword-only arguments
- `*args` / `**kwargs` without type hints
- Classes used where a function / module would suffice
- `__init__.py` with heavy logic / side effects
- Circular imports

## Code Smells
- Magic numbers / strings without constants
- `global` keyword usage
- Overly complex list / dict comprehensions
- Deep nesting where early return / guard clauses work
- `type()` checks instead of `isinstance()`
- `print` debugging left in source
- Commented-out code blocks
- `lambda` assigned to variables (use `def`)

## Best Practices
- Type hints for public function signatures and complex variables
- Context managers for resource management (`with` statements)
- Specific exception types (never bare `except:`)
- `pathlib.Path` over `os.path` for file operations
- f-strings over `.format()` or `%` formatting
- Proper `__init__.py` and module structure
- Docstrings for public API functions (Google / NumPy / reST style)
- `logging` module over `print` for diagnostics
- `dataclass(frozen=True)` / `NamedTuple` for immutable data
- `enum.Enum` / `StrEnum` over string constants
- Prefer `collections.abc` for type hints over concrete types
- `__all__` defined in modules for explicit exports
