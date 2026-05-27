# Benchmarking SAITS for Sparse Epidemiological Morbidity Data

This repository contains the code, data, and manuscript for a study that
benchmarks **SAITS** (Self-Attention-based Imputation for Time Series) against
seven conventional imputation methods on epidemiological morbidity data, and
then tests whether the quality of imputation carries through to **downstream
forecasting** with a Bidirectional LSTM.

> **Scope note.** SAITS is an existing architecture; this project does not
> propose a new imputation algorithm. The contribution is (1) the first
> systematic benchmark of SAITS on epidemiological data, (2) a structured
> missingness x disease evaluation framework, and (3) a downstream forecasting
> evaluation linking imputation quality to predictive accuracy.

---

## What the project does

1. **Loads raw morbidity data** for COVID-19 and for influenza / RSV.
2. **Pre-processes** it — including merging the duplicated weekly rows in the
   influenza data caused by the `ORIGIN_SOURCE` column.
3. **Creates artificial gaps** in three patterns — Point (scattered single
   gaps), Sequence (short runs), and Block (one long gap) — at 10%, 30%, and
   50% missing rates.
4. **Imputes** the gaps with eight methods: Mean, Median, Most-Frequent, LOCF,
   Linear interpolation, Rolling mean, MICE, and SAITS.
5. **Scores** the imputation with MAE, RMSE, KL-Divergence, and MASE.
6. **Forecasts** with a plain Bi-LSTM trained on each imputed series, scored
   with MAE and nRMSE, to check if better imputation gives better forecasts.

---

## Repository contents

### Notebook
| File | Description |
|------|-------------|
| `SAITS_Benchmark_V10.ipynb` | Main notebook — preprocessing, imputation benchmark, and forecasting. Run this top to bottom. |

### Raw input data (required to run the notebook)
| File | Description |
|------|-------------|
| `covid19_multi_mg_final.csv` | Daily COVID-19 case data, multiple countries. |
| `diseases_multi_mg_final.csv` | Weekly influenza / RSV surveillance data, multiple countries. |

### Generated result files
| File | Description |
|------|-------------|
| `benchmark_imputationV10.csv` | Imputation metrics (MAE, RMSE, KL-Divergence, MASE) per country, disease, pattern, rate, and method. |
| `imputed_series_for_forecasting.csv` | The full imputed time series (true vs imputed values, week by week) used as input to the forecaster. |
| `forecasting_results.csv` | Downstream Bi-LSTM forecasting results (MAE, nRMSE). |
| `Thesis_DMTests_V61.csv` | Diebold-Mariano statistical test results comparing forecast accuracy. |

### Figures
| File | Description |
|------|-------------|
| `Fig_Imputation_ScatteredGaps.png` | Line plot showing how each method fills scattered gaps in an influenza series. |
| `Fig_Forecasting_nRMSE_byDisease.png` | Forecasting nRMSE by disease and imputation method. |
| `Forecasting_MAE_comparison.png` | Forecasting MAE by disease and missingness pattern. |

### Manuscript
| File | Description |
|------|-------------|
| `PROCS-ICCSCI_2026_Nyein.tex` | LaTeX source of the paper (Elsevier `elsarticle` template). |

---

## How to run

### 1. Set up the environment
```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Check the raw data is present
The notebook expects `covid19_multi_mg_final.csv` and
`diseases_multi_mg_final.csv` in the same folder as the notebook (or update the
file paths in the CONFIGURATION cell).

### 3. Run the notebook
Open `SAITS_Benchmark_V10.ipynb` and run all cells in order. It will produce
the result CSVs and the figures listed above.

> **Note on runtime.** Training SAITS and the Bi-LSTM across all countries,
> patterns, and rates is computationally heavy and can take a long time on a
> CPU. A GPU (or Apple Silicon MPS) is recommended. The notebook auto-selects
> the device.

---

## Configuration options

The CONFIGURATION cell at the top of the notebook controls the experiment:

- **`COUNTRY_SELECTION_MODE`** — `"fixed"` uses a set list of countries for
  reproducible results; `"random"` picks qualifying countries at random to test
  generalizability.
- **`AGGREGATION_METHOD`** — how the duplicated weekly influenza rows
  (`ORIGIN_SOURCE`) are merged: `"sum"`, `"mean"`, or `"first"`.
- **`MISSING_RATES`**, **`PATTERNS`** — the missingness scenarios tested.
- **`FORECAST_HORIZONS`** — forecast length per disease (COVID-19 daily,
  influenza weekly).

---

## Key findings (summary)

- SAITS produced the most accurate gap reconstruction overall (lowest MAE and
  RMSE among the eight methods).
- Conventional constant-fill methods (Mean, Median, LOCF) collapse the temporal
  variance of the epidemic curve — the "flatline effect" — which inflates their
  KL-Divergence.
- In downstream forecasting, SAITS-imputed data gave the best forecasts for
  seasonal influenza and remained competitive for the more volatile COVID-19
  series, without ever degrading forecast accuracy catastrophically.

---

## Notes and limitations

- Exact reconstruction of a fully missing block is fundamentally limited for
  all methods, since no observed data exists inside the gap.
- The downstream Bi-LSTM forecaster is deliberately simple (no attention) so
  that forecasting differences reflect imputation quality, not forecaster
  capacity.
- Performance on low-count RSV series is weaker, as attention-based imputation
  benefits from a sufficient density of non-zero signal.
