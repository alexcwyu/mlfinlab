# MLFinLab

Machine Learning for Finance library by Hudson & Thames. Implements algorithms and techniques from "Advances in Financial Machine Learning" by Marcos Lopez de Prado, "Machine Learning for Asset Managers", and related research papers.

## Overview

MLFinLab provides reproducible, interpretable, and production-ready tools for portfolio managers and traders who want to leverage machine learning in quantitative finance. The library covers the full ML pipeline for financial data: from data structuring and feature engineering through labeling, sampling, model training, and bet sizing.

## Key Features

- **Data Structures**: Alternative bar types (tick, volume, dollar, imbalance, run bars) that provide better statistical properties than time bars
- **Labeling Methods**: Triple Barrier Method, meta-labeling, trend scanning, and multiple return-based labeling schemes
- **Feature Engineering**: Fractional differentiation for stationarity with memory preservation, microstructural features
- **Cross-Validation**: Combinatorial Purged Cross-Validation (CPCV) with embargo to prevent information leakage
- **Feature Importance**: MDI, MDA, SFI methods with clustered variants for correlated features
- **Structural Breaks**: SADF/GSADF tests, Chow test, CUSUM detection for regime changes
- **Bet Sizing**: Probability-based, dynamic, budget, and reserve bet sizing strategies
- **Sampling**: Sequential bootstrapping with concurrency-aware sample weights
- **Codependence**: Mutual information, variation of information, distance correlation, optimal transport
- **Clustering**: Optimal Number of Clusters (ONC) algorithm, hierarchical clustering, feature clusters
- **Networks**: MST, PMFG, ALMST graph construction from correlation/distance matrices
- **Data Generation**: Correlated random walks, CorrGAN, vine copulas, HCBM
- **Ensemble Methods**: Sequential bootstrapped bagging classifier
- **Backtest Statistics**: Deflated Sharpe ratio, probabilistic Sharpe ratio, strategy-level statistics

## Package Structure

```
src/mlfinlab/
    backtest_statistics/    # Strategy evaluation metrics
    bet_sizing/             # Position sizing from ML predictions
    clustering/             # ONC, hierarchical, feature clustering
    codependence/           # MI, VI, distance correlation, optimal transport
    cross_validation/       # CPCV, purged K-fold
    data_generation/        # Synthetic data (CorrGAN, vines, HCBM)
    data_structures/        # Alternative bar types
    datasets/               # Built-in sample datasets
    ensemble/               # Sequential bootstrapped bagging
    feature_importance/     # MDI, MDA, SFI (with clustering)
    features/               # Fractional differentiation
    filters/                # CUSUM filter, Z-score filter
    labeling/               # Triple Barrier, meta-labeling, trend scanning
    microstructural_features/ # Market microstructure feature generation
    multi_product/          # ETF trick for multi-asset
    networks/               # MST, PMFG, ALMST network graphs
    regression/             # History-weighted regression
    sample_weights/         # Attribution-based sample weighting
    sampling/               # Sequential bootstrapping, concurrency
    structural_breaks/      # SADF, Chow, CUSUM tests
    util/                   # EWMA, volatility, multiprocessing helpers
```

## Quick Start

This example generates synthetic tick data, creates dollar bars, applies CUSUM filtering and Triple Barrier labeling -- all with inline data, no downloads needed.

```python
import numpy as np
import pandas as pd
import mlfinlab as ml

# 1. Generate synthetic tick data (date_time, price, volume)
np.random.seed(42)
n_ticks = 50000
dates = pd.date_range("2020-01-01", periods=n_ticks, freq="s")
prices = 100 + np.cumsum(np.random.randn(n_ticks) * 0.01)
volumes = np.random.randint(1, 100, size=n_ticks).astype(float)
tick_data = pd.DataFrame({"date_time": dates, "price": prices, "volume": volumes})

# 2. Create dollar bars (samples when cumulative dollar value crosses threshold)
dollar_bars = ml.data_structures.get_dollar_bars(tick_data, threshold=50000, verbose=False)
print(f"Dollar bars created: {len(dollar_bars)} bars from {n_ticks} ticks")
print(dollar_bars[["open", "high", "low", "close", "cum_dollar_value"]].head())

# 3. Apply CUSUM filter to detect significant price moves
close = dollar_bars["close"]
daily_vol = ml.util.volatility.get_daily_vol(close, lookback=20)
cusum_events = ml.filters.cusum_filter(close, threshold=daily_vol.mean())
print(f"\nCUSUM events detected: {len(cusum_events)}")

# 4. Triple Barrier labeling
vertical_barriers = ml.labeling.add_vertical_barrier(cusum_events, close, num_days=5)
events = ml.labeling.get_events(
    close=close,
    t_events=cusum_events,
    pt_sl=[1, 1],
    target=daily_vol,
    min_ret=0.001,
    num_threads=1,
    vertical_barrier_times=vertical_barriers,
    verbose=False,
)
labels = ml.labeling.get_bins(events, close)
print(f"\nTriple Barrier labels distribution:\n{labels['bin'].value_counts()}")
```

## Dependencies

- numpy, pandas, scipy, scikit-learn, statsmodels
- matplotlib (visualization)
- networkx (graph algorithms)
- numba (JIT compilation for performance)

## Documentation

- [Architecture](architecture.md) — System design and components
- [Workflow](workflow.md) — Event flows and processing pipelines
- [State Management](state-management.md) — State lifecycle and data models
- [Development](development.md) — Development guide and best practices

## References

- Lopez de Prado, M. (2018). *Advances in Financial Machine Learning*. Wiley.
- Lopez de Prado, M. (2020). *Machine Learning for Asset Managers*. Cambridge University Press.
- Various research papers from Cornell and SSRN (see module docstrings for specific citations)
