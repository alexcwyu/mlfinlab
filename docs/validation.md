# Validation evidence: mlfinlab

Run date: 2026-10-12. Branch: `feature/ayu_develop`. Interpreter: `3.13.16 (main, Oct  9 2026, 15:11:24) [Clang 22.1.3 ]`.

| Check | Result |
|---|---|
| lock | passed |
| sync | passed |
| compile | passed |
| build | passed |
| tests | absent |

## Current result and compatibility decisions

- This public checkout contains API scaffolding with many pass bodies and no supplied tests. pytest exit5 confirms absence; package compilation and build do not certify research algorithms.
- TensorFlow 2.21.0 publishes neither a compatible macOS ARM64 Python3.14/3.15 wheel nor a source distribution. Managed Python3.13.16 is the latest runnable fallback.

Current unresolved checks:

- tests: No supplied test cases in this checkout; no cases were added.

## Latest full-suite output

```text
$ uv run --no-sync python -m pytest src/mlfinlab

no tests ran in 0.02s

```

## Exact commands and exit codes

Unsuccessful rows are retained as historical compatibility attempts; they are not claims that a current passing check remains blocked. Source-build probes use isolated temporary copies to preserve the selected repository environment.

| Command | Exit | Seconds | Evidence |
|---|---:|---:|---|
| `uv lock --upgrade --python 3.15` | 0 | 1.98 | [log](../../docs/maintenance-2026-10-12/mlfinlab-lock-3.15.log) |
| `uv sync --all-extras --all-groups --python 3.15 --no-build --no-install-project` | 2 | 0.05 | [log](../../docs/maintenance-2026-10-12/mlfinlab-wheel-probe-3.15.log) |
| `uv lock --upgrade --python 3.14` | 0 | 0.83 | [log](../../docs/maintenance-2026-10-12/mlfinlab-lock-3.14.log) |
| `uv sync --all-extras --all-groups --python 3.14 --no-build --no-install-project` | 2 | 0.04 | [log](../../docs/maintenance-2026-10-12/mlfinlab-wheel-probe-3.14.log) |
| `uv sync --all-extras --all-groups --python 3.14` | 2 | 0.05 | [log](../../docs/maintenance-2026-10-12/mlfinlab-sync-3.14.log) |
| `uv lock --upgrade --python 3.13` | 0 | 0.41 | [log](../../docs/maintenance-2026-10-12/mlfinlab-lock-3.13.log) |
| `uv sync --all-extras --all-groups --python 3.13 --no-build --no-install-project` | 2 | 0.05 | [log](../../docs/maintenance-2026-10-12/mlfinlab-wheel-probe-3.13.log) |
| `uv sync --all-extras --all-groups --python 3.13` | 0 | 2.24 | [log](../../docs/maintenance-2026-10-12/mlfinlab-sync-3.13.log) |
| `uv build --python 3.13` | 0 | 1.01 | [log](../../docs/maintenance-2026-10-12/mlfinlab-build.log) |
| `/Users/alex/.local/share/uv/python/cpython-3.13-macos-aarch64-none/bin/python3.13 -m compileall -q src` | 0 | 0.14 | [log](../../docs/maintenance-2026-10-12/mlfinlab-compile.log) |
| `uv run --no-sync python -m pytest` | 1 | 2.33 | [log](../../docs/maintenance-2026-10-12/mlfinlab-tests.log) |
| `uv run --no-sync python -m pytest src/mlfinlab` | 5 | 1.61 | [log](../../docs/maintenance-2026-10-12/mlfinlab-tests-existing-source.log) |
| `uv run --no-sync python -m compileall -q src` | 0 | 0.13 | [log](../../docs/maintenance-2026-10-12/mlfinlab-compile-final.log) |
| `uv build --python 3.13` | 0 | 1.05 | [log](../../docs/maintenance-2026-10-12/mlfinlab-build-final.log) |
| `uv run --no-sync python -c import sys,importlib.metadata,json; print(sys.version); print(json.dumps({d.metadata["Name"]: d.version for d in importlib.metadata.distributions()},indent=2,sort_keys=True))` | 0 | 0.24 | [log](../../docs/maintenance-2026-10-12/mlfinlab-versions-final.log) |
| `uv sync --directory /tmp/trading-python315-probes/mlfinlab --all-extras --all-groups --python 3.15` | 2 | 0.66 | [log](../../docs/maintenance-2026-10-12/mlfinlab-python315-source-probe.log) |

The versions-final log records the exact interpreter patch release and every installed dependency version. The machine-readable [result](../../docs/maintenance-2026-10-12/mlfinlab-result.json) includes branch/default ancestry, status, artifacts and unresolved checks.

## Dependency floor refresh

| Before | After |
|---|---|
| `hatchling` | `hatchling>=1.32.4` |
| `cython>=3.2.1` | `cython>=3.3.0` |
| `numpy>=2.3.4` | `numpy>=2.5.4` |
| `matplotlib>=3.10.7` | `matplotlib>=3.11.2` |
| `pandas>=2.3.3` | `pandas>=3.0.6` |
| `scikit-learn>=1.7.2` | `scikit-learn>=1.9.1` |
| `scipy>=1.16.3` | `scipy>=1.18.1` |
| `statsmodels>=0.14.5` | `statsmodels>=0.15.0` |
| `POT>=0.9.6` | `POT>=0.9.7.post1` |
| `numba>=0.62.1` | `numba>=0.68.0` |
| `networkx>=3.5` | `networkx>=3.7` |
| `dash>=3.3.0` | `dash>=4.4.1` |
| `tensorflow>=2.20.0` | `tensorflow>=2.21.0` |
| `joblib>=1.5.2` | `joblib>=1.6.0` |
| `decorator>=5.1.1` | `decorator>=5.3.1` |
| `pytest>=9.0` | `pytest>=9.1.1` |
| `pytest-cov>=6.0.0` | `pytest-cov>=7.1.0` |
| `coverage>=7.6.10` | `coverage>=7.16.2` |
| `pylint>=3.3.5` | `pylint>=4.1.3` |
| `sphinx>=8.1.3` | `sphinx>=9.1.0` |
| `sphinx-rtd-theme>=3.0.2` | `sphinx-rtd-theme>=3.1.0` |

The table records the initial metadata refresh. Later compatibility constraints and vendor sources are explained above and the final authoritative requirements are in [pyproject.toml](../pyproject.toml) and [uv.lock](../uv.lock). No test cases were added, removed or disabled. Existing deprecated-API cases retain their assertions while explicitly checking the documented warning.
