# py-compat-scan

[![Python 3.8+](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/)
[![Python 3.11](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/)
[![Python 3.12](https://img.shields.io/badge/python-3.12-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![CI](https://github.com/toxicwind/py-compat-scan/actions/workflows/ci.yml/badge.svg)](https://github.com/toxicwind/py-compat-scan/actions)

> **AST-based compatibility scanner for Python 3.12+ → 3.11/3.8 fallback detection.**
>
> Zero runtime execution. Pure static analysis via `ast` module. Identifies syntax constructs introduced in Python 3.9–3.12 that break on 3.8 or 3.11, with suggested rewrites.

## What It Detects

| Feature | 3.12+ Syntax | Fallback | Severity |
|---------|-------------|----------|----------|
| `type` keyword (PEP 695) | `type Point[T] = tuple[T, T]` | `typing.TypeVar` + `typing.Generic` | 🔴 Hard |
| `except*` groups (PEP 654) | `except* ValueError:` | Nested `try/except` with group tracking | 🔴 Hard |
| `f-string` debug `=` (3.8) | `f"{x=}"` | `f"x={x}"` | 🟡 Soft |
| `match/case` (3.10) | `match obj:` | `if/elif` chain or `structural-pattern-matching` polyfill | 🟡 Soft |
| `tomllib` (3.11) | `import tomllib` | `import tomli; tomli.loads()` | 🟡 Soft |
| `typing.ParamSpec` (3.10) | `**P` | `typing_extensions.ParamSpec` | 🟡 Soft |
| `ast` unparse (3.9) | `ast.unparse(node)` | `astor` or manual codegen | 🟡 Soft |

## Install

```bash
pip install py-compat-scan
```

## Usage

```bash
# Scan a single file
py-compat-scan --target 3.8 src/my_module.py

# Scan a package recursively
py-compat-scan --target 3.11 --recursive ./src

# Output JSON for CI gates
py-compat-scan --target 3.8 --format json --fail-on hard ./src
```

## API

```python
from py_compat_scan import Scanner

scanner = Scanner(target="3.8")
results = scanner.scan_path("./src")
for issue in results.hard_blocks:
    print(f"{issue.file}:{issue.line} → {issue.feature} requires {issue.min_version}")
```

## CI Integration

```yaml
- uses: toxicwind/py-compat-scan@v1
  with:
    target: "3.8"
    fail-on: "hard"
    paths: "./src"
```

## Architecture

```
py_compat_scan/
├── __init__.py
├── scanner.py          # AST visitor + version matrix
├── features.py         # Feature registry (PEP → version → fallback)
├── rewriters.py        # Suggested code transformations
└── cli.py              # argparse + json/yaml output
```

## License

MIT — see [LICENSE](./LICENSE).
