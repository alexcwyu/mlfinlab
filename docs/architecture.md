# MLFinLab Architecture

## System Architecture Overview

MLFinLab follows a modular, pipeline-oriented architecture where each module addresses a specific stage of the financial machine learning workflow. Modules are designed to be used independently or composed into complete pipelines.

```mermaid
graph TB
    subgraph "Data Ingestion"
        RAW[Raw Tick Data] --> DS[Data Structures]
        DS --> BARS[Alternative Bars]
    end

    subgraph "Feature Engineering"
        BARS --> FE[Features / FracDiff]
        BARS --> MSF[Microstructural Features]
        BARS --> COD[Codependence Measures]
    end

    subgraph "Labeling & Sampling"
        BARS --> LAB[Labeling Module]
        LAB --> SW[Sample Weights]
        LAB --> SAMP[Sampling / Bootstrap]
    end

    subgraph "Model Training"
        FE --> CV[Cross-Validation]
        MSF --> CV
        SW --> CV
        SAMP --> CV
        CV --> ENS[Ensemble Methods]
        CV --> FI[Feature Importance]
    end

    subgraph "Strategy Evaluation"
        ENS --> BS[Bet Sizing]
        FI --> BS
        BS --> BTS[Backtest Statistics]
        BARS --> SB[Structural Breaks]
    end

    subgraph "Network Analysis"
        COD --> NET[Networks / Graphs]
        COD --> CLU[Clustering]
    end
```

## Trading Paradigm & Key Features

| Feature | Support | Details |
|---------|---------|---------|
| Backtesting Approach | None | Research library only; provides backtest statistics but no backtesting engine |
| Live Trading | No | No execution or order management capabilities |
| Paper Trading | No | Not applicable -- research/analysis library |
| Multi-Asset | Yes | Asset-agnostic; operates on pandas DataFrames of any asset class |
| Data Feeds | None | Accepts pandas DataFrames/Series and CSV files as input |
| ML Integration | Yes | Core purpose -- implements ML pipeline from AFML book (labeling, CV, feature importance, ensemble methods) |
| Risk Management | Custom | Bet sizing module converts ML predictions to position sizes |
| Optimization | No | No built-in optimization; designed to feed into external ML model training |
| Execution | None | No execution capabilities; outputs labels, features, and position sizes for external use |

## Core Design Patterns

### Abstract Base Class Hierarchy (Data Structures)

The bar generation system uses a layered class hierarchy:

```mermaid
classDiagram
    class BaseBars {
        <<abstract>>
        +batch_run(file_path_or_df)
        +run(data)
        #_extract_bars(data)*
        #_reset_cache()*
        #_apply_tick_rule(price)
        #_get_imbalance(price, signed_tick, volume)
        #_create_bars(date_time, price, high, low, list_bars)
    }

    class BaseImbalanceBars {
        <<abstract>>
        #_extract_bars(data)
        #_reset_cache()
        #_get_expected_imbalance(window)
        #_get_exp_num_ticks()*
    }

    class BaseRunBars {
        <<abstract>>
        #_extract_bars(data)
        #_reset_cache()
        #_get_expected_imbalance(array, window)
        #_get_exp_num_ticks()*
    }

    class StandardBars {
        +get_tick_bars()
        +get_volume_bars()
        +get_dollar_bars()
    }

    class ImbalanceDataStructures {
        +get_ema_dollar_imbalance_bars()
        +get_const_dollar_imbalance_bars()
    }

    class RunDataStructures {
        +get_ema_dollar_run_bars()
        +get_const_dollar_run_bars()
    }

    BaseBars <|-- BaseImbalanceBars
    BaseBars <|-- BaseRunBars
    BaseBars <|-- StandardBars
    BaseImbalanceBars <|-- ImbalanceDataStructures
    BaseRunBars <|-- RunDataStructures
```

### Multiprocessing Architecture

MLFinLab uses a parallelization utility (`mp_pandas_obj`) throughout the library for computationally intensive operations:

```mermaid
flowchart LR
    INPUT[Input DataFrame] --> SPLIT[Split by molecule]
    SPLIT --> W1[Worker 1]
    SPLIT --> W2[Worker 2]
    SPLIT --> WN[Worker N]
    W1 --> MERGE[Merge Results]
    W2 --> MERGE
    WN --> MERGE
    MERGE --> OUTPUT[Output DataFrame/Series]
```

Key uses:
- Triple Barrier labeling (parallelized across event timestamps)
- SADF computation (parallelized across time indices)
- Cross-validation scoring (parallelized across folds)
- Sample weight calculation (parallelized across observations)

### Labeling Pipeline

The labeling system follows a sequential pipeline pattern:

```mermaid
flowchart TD
    CLOSE[Close Prices] --> FILTER[CUSUM / Z-score Filter]
    FILTER --> EVENTS[t_events timestamps]
    EVENTS --> VB[add_vertical_barrier]
    CLOSE --> VOL[Daily Volatility]
    VOL --> GE[get_events]
    EVENTS --> GE
    VB --> GE
    GE --> TB_EVENTS[Triple Barrier Events]
    TB_EVENTS --> GB[get_bins]
    CLOSE --> GB
    GB --> LABELS[Labeled Samples]
    LABELS --> DL[drop_labels - remove rare]
```

### Cross-Validation with Purging and Embargo

```mermaid
flowchart LR
    subgraph "Standard K-Fold"
        S1[Split 1] --> S2[Split 2] --> S3[Split 3] --> S4[Split 4]
    end

    subgraph "Purged K-Fold"
        PS1[Split 1] --> PURGE[Purge Overlapping] --> EMB[Apply Embargo]
        EMB --> PS2[Clean Train Set]
    end

    subgraph "Combinatorial PCV"
        COMB[All C(N,K) combinations] --> PATHS[Backtest Paths]
        PATHS --> EVAL[Path-wise Evaluation]
    end
```

## Module Dependencies

```mermaid
graph TD
    UTIL[util] --> DS[data_structures]
    UTIL --> LAB[labeling]
    UTIL --> MSF[microstructural_features]
    UTIL --> SADF[structural_breaks]

    DS --> LAB
    DS --> MSF

    LAB --> CV[cross_validation]
    LAB --> SW[sample_weights]

    CV --> FI[feature_importance]
    CV --> ENS[ensemble]

    FI --> BS[bet_sizing]

    COD[codependence] --> CLU[clustering]
    COD --> NET[networks]
```

## Key Architectural Decisions

1. **Stateless functions over classes**: Most modules expose top-level functions rather than requiring class instantiation, following the pattern in the AFML book.

2. **Pandas-native interfaces**: All inputs and outputs use pandas Series/DataFrame, ensuring interoperability with the Python data science ecosystem.

3. **Batch processing for large datasets**: The `BaseBars` class processes tick data in configurable batch sizes (default 20M rows) to handle datasets that do not fit in memory.

4. **Abstract methods for extensibility**: The bar system uses abstract methods (`_extract_bars`, `_reset_cache`) to allow new bar types to be added by implementing a few methods.

5. **EWMA-based adaptive thresholds**: Information-driven bars (imbalance and run bars) use exponentially weighted moving averages to adaptively set bar formation thresholds.

6. **Separation of concerns in labeling**: The labeling pipeline separates event detection (filters), barrier construction (get_events), and label assignment (get_bins) into distinct, composable steps.

---
## See Also
- [README](README.md) — Project overview and quick start
- [Workflow](workflow.md) — Event flows and processing pipelines
- [State Management](state-management.md) — State lifecycle and data models
- [Development](development.md) — Development guide and best practices
