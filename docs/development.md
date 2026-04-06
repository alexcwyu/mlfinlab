# MLFinLab Development Guide

## Project Setup

```bash
cd mlfinlab
pip install -e .
# or
pip install -e ".[dev]"
```

## Dependencies

**Core**:
- numpy, pandas, scipy
- scikit-learn (cross-validation, clustering, metrics)
- statsmodels (ADF tests, statistical models)
- matplotlib (plotting)
- networkx (graph algorithms)
- numba (JIT compilation for EWMA)

**Optional**:
- dash, plotly (interactive network visualizations)
- cvxopt (optimization)

## Project Structure

```
mlfinlab/
    src/mlfinlab/           # Main package source
        __init__.py         # Top-level imports for all modules
        backtest_statistics/
        bet_sizing/
        clustering/
        codependence/
        cross_validation/
        data_generation/
        data_structures/
        datasets/
        ensemble/
        feature_importance/
        features/
        filters/
        labeling/
        microstructural_features/
        multi_product/
        networks/
        regression/
        sample_weights/
        sampling/
        structural_breaks/
        util/
```

## Code Conventions

### Naming

- Module-level functions use `snake_case` (e.g., `get_sadf`, `get_mutual_info`)
- Classes use `PascalCase` (e.g., `BaseBars`, `CombinatorialPurgedKFold`)
- Private/helper functions prefixed with `_` (e.g., `_get_sadf_at_t`)
- Constants are UPPER_CASE

### Docstrings

All public functions follow the reStructuredText docstring format with AFML references:

```python
def function_name(param1, param2):
    """
    Advances in Financial Machine Learning, Snippet X.Y, page Z.

    Brief description of what the function does.

    :param param1: (type) Description of param1
    :param param2: (type) Description of param2
    :return: (type) Description of return value
    """
```

### Type Hints

Functions use type hints from `typing` module:

```python
from typing import Tuple, Union, Optional
def func(series: pd.Series, model: str, lags: Union[int, list]) -> pd.Series:
```

## Testing

```bash
# Run all tests
pytest

# Run specific module tests
pytest tests/test_labeling.py
pytest tests/test_data_structures.py

# With coverage
pytest --cov=mlfinlab
```

### Test Data

The `datasets` module provides built-in sample data:

```python
from mlfinlab.datasets import load_datasets
tick_data = load_datasets.load_tick_sample()
```

## Adding a New Module

1. Create a new directory under `src/mlfinlab/` with `__init__.py`
2. Implement functions following the existing patterns:
   - Accept pandas Series/DataFrame inputs
   - Return pandas Series/DataFrame outputs
   - Include AFML book references in docstrings
   - Support multiprocessing via `mp_pandas_obj` where applicable
3. Add the module import to `src/mlfinlab/__init__.py`
4. Add tests under `tests/`

## Key Implementation Patterns

### Multiprocessing Helper

Use `mp_pandas_obj` for parallelizable computations:

```python
from mlfinlab.util.multiprocess import mp_pandas_obj

result = mp_pandas_obj(
    func=_inner_function,
    pd_obj=('molecule', events.index),
    num_threads=num_threads,
    # additional kwargs passed to _inner_function
    close=close,
    events=events
)
```

### Batch Processing for Large Data

Follow the `BaseBars` pattern for streaming large datasets:

```python
def batch_run(self, file_path_or_df, batch_size=2e7):
    for batch in self._batch_iterator(file_path_or_df):
        bars = self._extract_bars(batch)
        # accumulate bars
```

### EWMA Utility

For adaptive threshold computation:

```python
from mlfinlab.util.fast_ewma import ewma
smoothed = ewma(array, window=100)
```

## Performance Considerations

- **Memory**: Bar generation processes data in chunks; set `batch_size` appropriately for your memory constraints
- **CPU**: Use `num_threads` parameter on parallelized functions; defaults vary by function
- **Numba**: The `fast_ewma` module uses Numba JIT compilation for significant speedup
- **Large datasets**: Use the `to_csv=True` flag in `batch_run()` to stream results to disk

## Common Development Tasks

### Implementing a New Bar Type

1. Subclass `BaseBars` (or `BaseImbalanceBars` / `BaseRunBars`)
2. Implement `_extract_bars(data)` -- the core bar formation logic
3. Implement `_reset_cache()` -- what state to reset when a new bar forms
4. Create a module-level factory function (e.g., `get_my_bars()`)
5. Export from `src/mlfinlab/data_structures/__init__.py`

### Implementing a New Labeling Method

1. Add a new file under `labeling/`
2. Implement a function that accepts price data and returns labeled DataFrame
3. Export from `src/mlfinlab/labeling/__init__.py`
4. Follow the convention: return DataFrame with columns appropriate to the method

### Implementing a New Codependence Measure

1. Add function to `codependence/` module
2. Accept `x: np.array, y: np.array` inputs
3. Return a float score
4. Support optional `normalize` parameter for [0,1] scaling
5. Export from `src/mlfinlab/codependence/__init__.py`

## Configuration Reference

MLFinLab functions are configured via function parameters rather than global configuration files. Below are the key parameters for the most commonly used modules.

### Dollar / Volume / Tick Bars (`data_structures`)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `file_path_or_df` | str, list, or DataFrame | _(required)_ | Path to CSV or DataFrame with columns `[date_time, price, volume]` |
| `threshold` | float or Series | `70000000` | Cumulative value threshold to trigger a bar sample |
| `batch_size` | int | `20000000` | Number of rows per batch (lower = less RAM) |
| `verbose` | bool | `True` | Print batch progress to console |
| `to_csv` | bool | `False` | Stream results to CSV after each batch |
| `output_path` | str | `None` | CSV output path when `to_csv=True` |

### Triple Barrier Labeling (`labeling.get_events`)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `close` | Series | _(required)_ | Close price series |
| `t_events` | Series | _(required)_ | Timestamps from CUSUM filter or other event detector |
| `pt_sl` | list[float] | _(required)_ | `[profit_taking, stop_loss]` width multipliers (0 disables barrier) |
| `target` | Series | _(required)_ | Daily volatility series (used to scale barriers) |
| `min_ret` | float | _(required)_ | Minimum return threshold for a label to be non-zero |
| `num_threads` | int | _(required)_ | Number of parallel threads for computation |
| `vertical_barrier_times` | Series or False | `False` | Timestamps of vertical barriers from `add_vertical_barrier()` |
| `side_prediction` | Series | `None` | Side predictions for meta-labeling (1 = long, -1 = short) |
| `verbose` | bool | `True` | Print progress |

### CUSUM Filter (`filters.cusum_filter`)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `raw_time_series` | Series | _(required)_ | Input time series (typically close prices) |
| `threshold` | float | _(required)_ | Cumulative deviation threshold to trigger a sample |
| `time_stamps` | bool | `True` | Return timestamps (`True`) or boolean mask (`False`) |

### Fractional Differentiation (`fracdiff.frac_diff_ffd`)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `series` | DataFrame | _(required)_ | Input price series |
| `diff_amt` | float | _(required)_ | Fractional differentiation order (0 < d < 1) |
| `thresh` | float | `1e-5` | Weight cutoff threshold for the FFD filter |

### Combinatorial Purged K-Fold (`cross_validation.CombinatorialPurgedKFold`)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `n_splits` | int | _(required)_ | Total number of folds |
| `n_test_splits` | int | _(required)_ | Number of folds used for testing in each combination |
| `samples_info_sets` | Series | _(required)_ | Maps each sample to its information set end time (for purging) |
| `pct_embargo` | float | `0.0` | Percentage of data to embargo after each test set |

### Daily Volatility (`util.volatility.get_daily_vol`)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `close` | Series | _(required)_ | Close price series |
| `lookback` | int | `100` | Lookback window for exponentially weighted std of returns |

### Sequential Bootstrap (`sampling.get_ind_matrix` / `sampling.get_ind_mat_label_uniqueness`)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `samples_info_sets` | Series | _(required)_ | Information set end times per sample |
| `price_bars` | DataFrame | _(required)_ | Price bar DataFrame with DatetimeIndex |

### Multiprocessing Helper (`util.multiprocess.mp_pandas_obj`)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `func` | callable | _(required)_ | Function to parallelize |
| `pd_obj` | tuple | _(required)_ | `('molecule', index)` tuple defining the parallelization axis |
| `num_threads` | int | `1` | Number of parallel threads (1 = sequential) |
| `mp_batches` | int | `1` | Number of batches per thread for load balancing |
| `lin_parts` | bool | `True` | Use linear partitioning (`True`) or nested partitioning (`False`) |

## Troubleshooting

### 1. `ValueError: Expected columns [date_time, price, volume]` in bar generation
**Cause**: Input DataFrame columns do not match the expected format.
**Solution**: Ensure your tick data has exactly three columns named `date_time`, `price`, and `volume`. The `date_time` column must be parseable as datetime.

### 2. `MemoryError` when generating bars from large datasets
**Cause**: The entire dataset is loaded into memory at once.
**Solution**: Reduce `batch_size` parameter (e.g., `batch_size=1_000_000`). Use `to_csv=True` with `output_path` to stream results to disk. Pass a file path instead of a DataFrame to enable batch processing.

### 3. `get_events` returns empty DataFrame
**Cause**: No events triggered -- either `t_events` is empty, `min_ret` is too high, or barriers are too narrow.
**Solution**: Check that `t_events` contains timestamps. Lower `min_ret`, widen `pt_sl` multipliers, or increase the `threshold` in `cusum_filter` to get more event samples.

### 4. `ZeroDivisionError` in `get_daily_vol`
**Cause**: Input series has too few data points or all identical prices (zero returns).
**Solution**: Ensure at least `lookback + 1` data points. Verify prices are not constant.

### 5. Numba JIT compilation fails (`numba.core.errors.TypingError`)
**Cause**: Numba cannot compile the fast EWMA function, often due to incompatible Numba/NumPy versions.
**Solution**: Upgrade Numba and NumPy to compatible versions (`pip install numba numpy --upgrade`). The `fast_ewma` module requires Numba JIT support.

### 6. `mp_pandas_obj` hangs or produces no output
**Cause**: Deadlock in multiprocessing, often on macOS or in Jupyter notebooks.
**Solution**: Set `num_threads=1` to disable multiprocessing. On macOS, set `export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES` before running.

### 7. Triple Barrier labels are all zeros
**Cause**: The `target` (volatility) series values are much larger than actual price moves, making barriers unreachable.
**Solution**: Check the scale of `target` values relative to price changes. Use `get_daily_vol()` with an appropriate `lookback` to compute properly scaled volatility.

### 8. `KeyError` or `IndexError` in cross-validation
**Cause**: `samples_info_sets` index does not align with the training data index.
**Solution**: Ensure `samples_info_sets` has the same DatetimeIndex as your feature matrix. All indices must be present in both.

## Security Considerations

### API Key Management
- MLFinLab is a research library and does not directly connect to exchanges or APIs. However, data pipelines feeding into MLFinLab may use API keys.
- Keep API keys for data providers (e.g., for fetching tick data from exchanges) in environment variables, not in notebooks or scripts.

### Credential Storage
- Sample datasets are bundled in `mlfinlab/datasets/data/`. These contain no sensitive data.
- If loading proprietary datasets from databases or cloud storage, use environment-based credential injection rather than hardcoded connection strings.

### Network Security
- MLFinLab performs all computation locally; it makes no outbound network calls.
- When using `mp_pandas_obj` for multiprocessing, all processes run locally on the same machine. No data leaves the host.

### Safe Practices
- Validate input data before passing to labeling functions. Malformed tick data (negative prices, NaN volumes) can produce misleading labels.
- When writing bar data to CSV with `to_csv=True`, ensure the output directory has appropriate permissions to prevent unauthorized access to trading data.
- Use `verbose=False` in production pipelines to avoid leaking data details to log files.
- Pin dependency versions (NumPy, Numba, pandas) to avoid breaking changes in JIT-compiled functions.
- Be cautious with `num_threads` in shared computing environments; excessive parallelism can affect other users.

---
## See Also
- [README](README.md) — Project overview and quick start
- [Architecture](architecture.md) — System design and components
- [Workflow](workflow.md) — Event flows and processing pipelines
- [State Management](state-management.md) — State lifecycle and data models
