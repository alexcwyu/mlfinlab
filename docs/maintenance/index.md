# mlfinlab: portable verification records

Full raw logs are retained in the workspace maintenance evidence, not distributed here. Hashes identify those original logs. A successful build or selected test subset does not imply a full test-suite pass. Commands containing <workspace> describe the original execution environment.

[Machine-readable records](verification.json) · [Repository validation summary](../validation.md)

## Executed commands

| Check | Exit | Duration | Command | Outcomes |
| --- | ---: | ---: | --- | --- |
| build | 0 | 1.01s | `uv build --python 3.13` | Successfully built dist/mlfinlab-1.3.0.tar.gz<br>Successfully built dist/mlfinlab-1.3.0-py3-none-any.whl |
| build-final | 0 | 1.05s | `uv build --python 3.13` | Successfully built dist/mlfinlab-1.3.0.tar.gz<br>Successfully built dist/mlfinlab-1.3.0-py3-none-any.whl |
| compile | 0 | 0.14s | `/Users/alex/.local/share/uv/python/cpython-3.13-macos-aarch64-none/bin/python3.13 -m compileall -q src` | See exit status; no test-count assertion. |
| compile-final | 0 | 0.13s | `uv run --no-sync python -m compileall -q src` | See exit status; no test-count assertion. |
| lock-3.13 | 0 | 0.41s | `uv lock --upgrade --python 3.13` | See exit status; no test-count assertion. |
| lock-3.14 | 0 | 0.83s | `uv lock --upgrade --python 3.14` | See exit status; no test-count assertion. |
| lock-3.15 | 0 | 1.98s | `uv lock --upgrade --python 3.15` | See exit status; no test-count assertion. |
| python315-source-probe | 2 | 0.66s | `uv sync --directory /tmp/trading-python315-probes/mlfinlab --all-extras --all-groups --python 3.15` | See exit status; no test-count assertion. |
| sync-3.13 | 0 | 2.24s | `uv sync --all-extras --all-groups --python 3.13` | See exit status; no test-count assertion. |
| sync-3.14 | 2 | 0.05s | `uv sync --all-extras --all-groups --python 3.14` | See exit status; no test-count assertion. |
| tests | 1 | 2.33s | `uv run --no-sync python -m pytest` | See exit status; no test-count assertion. |
| tests-existing-source | 5 | 1.61s | `uv run --no-sync python -m pytest src/mlfinlab` | See exit status; no test-count assertion. |
| versions-final | 0 | 0.24s | `uv run --no-sync python -c import sys,importlib.metadata,json; print(sys.version); print(json.dumps({d.metadata["Name"]: d.version for d in importlib.metadata.distributions()},indent=2,sort_keys=True))` | See exit status; no test-count assertion. |
| wheel-probe-3.13 | 2 | 0.05s | `uv sync --all-extras --all-groups --python 3.13 --no-build --no-install-project` | See exit status; no test-count assertion. |
| wheel-probe-3.14 | 2 | 0.04s | `uv sync --all-extras --all-groups --python 3.14 --no-build --no-install-project` | See exit status; no test-count assertion. |
| wheel-probe-3.15 | 2 | 0.05s | `uv sync --all-extras --all-groups --python 3.15 --no-build --no-install-project` | See exit status; no test-count assertion. |
