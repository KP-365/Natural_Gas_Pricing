```markdown
# Natural Gas Storage Forecasting with Calibrated Uncertainty

## Project Overview

This project implements a **calibrated-uncertainty forecasting pipeline** for weekly U.S. natural gas storage change within a single Jupyter Notebook. The system compares a point-estimate baseline against two uncertainty-aware models, then tests whether that predicted uncertainty carries genuine predictive value for realised natural gas price volatility, framed as an alpha-seeking exercise structurally identical to how a candidate trading signal would be validated. The study includes a **225-week walk-forward backtest** to validate model accuracy and calibration under real, out-of-sample market conditions.

The notebook contains three core forecasting models:
- **Baseline Model** (OLS regression): point-estimate accuracy floor, no uncertainty
- **Bayesian Regression** (PyMC, NUTS): uncertainty from a full posterior distribution over parameters
- **Heteroscedastic Neural Network** (PyTorch): uncertainty learned directly as a function of the input

Additionally, the notebook provides a Thompson sampling adaptive decision layer, robustness checks against known financial confounders, and a practical deliverable (a linear and logistic regression model with an entropy-gated confidence threshold) built on whichever factors survive rigorous testing.

---

## Table of Contents

- [System Requirements](#system-requirements)
- [Dependencies](#dependencies)
- [Installation & Setup](#installation--setup)
- [Notebook Structure](#notebook-structure)
- [Dataset Description](#dataset-description)
- [Running Guide](#running-guide)
- [Outputs & Results](#outputs--results)
- [Citation & Acknowledgements](#citation--acknowledgements)

---

## System Requirements

- **Python**: 3.12
- **Operating System**: Windows 10/11, macOS, or Linux (or Google Colab, no local setup required)
- **Memory**: 4GB+ recommended
- **GPU**: Not required; all models fit and backtest within a few minutes on standard CPU compute
- **Jupyter Environment**: Jupyter Notebook, JupyterLab, or Google Colab

---

## Dependencies

Install the following packages before running the notebook:

```text
numpy>=1.21.0
pandas>=1.3.0
requests>=2.26.0
pymc>=6.2.0
torch>=2.0.0
statsmodels>=0.13.0
scikit-learn>=1.0.0
scipy>=1.7.0
matplotlib>=3.4.0
```

> **Note**: `pymc` and `torch` are required, not optional, both the Bayesian regression and the heteroscedastic neural network depend on them directly.

---

## Installation & Setup

### 1. Create a Virtual Environment (Recommended)

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS / Linux
python3 -m venv venv
source venv/bin/activate
```

### 2. Install Dependencies

Save the dependency list above as `requirements.txt`, then run:

```bash
pip install -r requirements.txt
```

Or install directly:

```bash
pip install numpy pandas requests pymc torch statsmodels scikit-learn scipy matplotlib
```

### 3. Launch Jupyter

```bash
jupyter notebook
```

Alternatively, upload `storage_volatility_final_analysis.ipynb` directly to Google Colab, no local installation is required there.

### 4. Prepare the Data

No local data files are needed. All storage, price, capacity, and weather data is fetched live from free public APIs when the notebook runs (see Dataset Description below). An EIA API key is already set in the configuration cell for marking convenience; no setup is required to run the notebook as submitted.

---

## Notebook Structure

The entire project is contained in **one Jupyter Notebook** (`storage_volatility_final_analysis.ipynb`), organised into an eight-stage pipeline:

| Stage | Description |
|---|---|
| **Stage 1: Data Ingestion** | Fetches EIA storage level, EIA price, EIA capacity, and NOAA HDD/CDD; derives storage change; chronological train/validation/test split |
| **Stage 2: Forecasting Models** | Fits the baseline (OLS), Bayesian regression (PyMC/NUTS), and heteroscedastic neural network (PyTorch); compares RMSE and coverage on the held-out test set |
| **Stage 3: Volatility Validation** | Initial check on the single test split, correlates predicted interval width against realised Henry Hub volatility |
| **Stage 4: CRPS** | Computes the Continuous Ranked Probability Score for both uncertainty-aware models |
| **Stage 5: Walk-Forward Backtest** | Rolling-origin backtest refitting each model at every step across 225 out-of-sample weeks |
| **Stage 6: Thompson Sampling** | Adaptive hedging-policy decision layer selecting between aggressive, conservative, and forecast-following policies |
| **Stage 7: Robustness Checks** | Joint regression controlling for naive persistence and price momentum; outlier robustness check |
| **Stage 8: Practical Deliverable** | Linear and logistic regression models trained on the factors shown to survive Stage 7, with an entropy-gated confidence threshold on the classifier |

---

## Dataset Description

### Data Source

This project uses **free, public U.S. energy and weather data**, fetched live via API, no data files are bundled with this submission.

- **Data Type**: Public weekly storage, daily price, and weekly weather data
- **Content**: U.S. Lower-48 natural gas storage levels, Henry Hub spot prices, working gas capacity, and national population-weighted HDD/CDD, August 2020 to present
- **Acquisition**: Fetched programmatically via the following free APIs
  - EIA Weekly Natural Gas Storage Report: https://www.eia.gov/naturalgas/storage/dashboard/ (v2 API, `natural-gas/stor/wkly`)
  - EIA Natural Gas Futures Prices: v2 API, `natural-gas/pri/fut`
  - EIA Underground Natural Gas Storage Capacity: v2 API, `natural-gas/stor/cap`
  - NOAA Climate Prediction Center: https://www.ncdc.noaa.gov/cdo-web/ (free plain-text files, no key required)
- **Format Requirements**: None, all data is fetched and parsed automatically by the notebook's data-ingestion functions

---

## Running Guide

Execute the notebook **sequentially from top to bottom**. Do not skip cells, later stages depend on variables defined in earlier ones.

### Step-by-Step Execution

```python
# Stage 1: Set the EIA API key (already provided) and run the data-ingestion cells
EIA_API_KEY = "provided-key-here"

# Stages 2-4: Fit and evaluate the three forecasting models on the single test split

# Stage 5: Run the walk-forward backtest (this may take a minute)
wf_results = walk_forward_backtest(X_all, y_all, storage.index, min_train=80)

# Stage 6: Run the Thompson sampling decision layer

# Stage 7: Run the robustness checks (naive persistence, joint regression, outlier test)

# Stage 8: Train and validate the practical deliverable models
```

### Key Execution Notes

- **Do not skip Stage 1**; every later stage depends on the `storage`, `price`, `hdd`, `cdd`, and `storage_fraction` variables it defines
- The walk-forward backtest (Stage 5) is the most computationally expensive step, expect a few minutes to run
- If the EIA API key stops working, register a new free key instantly at https://www.eia.gov/opendata/register.php and replace it in the configuration cell

---

## Outputs & Results

After executing the notebook, the following results are generated inline (no external files are written):

| Output | Section | Description |
|---|---|---|
| Table 1 | Stage 2 | RMSE and 90% coverage for all three models, single-split and walk-forward |
| Table 2 | Stage 4 | CRPS for the Bayesian regression and heteroscedastic NN |
| Table 3 | Stage 7 | Raw correlations of all tested predictors with realised volatility |
| Figure 1 | Stage 5 | Walk-forward backtest plot: actual vs. baseline vs. Bayesian vs. NN forecasts |
| Figure 2 | Stage 8 | Prediction confidence vs. correctness breakdown for the entropy-gated classifier |
| Printed statistics | Throughout | Correlation coefficients, p-values, and R² reported directly in cell output |

### Visualisations Generated

- **Walk-Forward Backtest Plot**: actual storage change vs. all three models' forecasts, with the Bayesian 90% interval shaded
- **Confidence Breakdown Chart**: confident-vs-unsure, correct-vs-wrong predictions for the entropy-gated binary classifier
- **Thompson Sampling Plots**: policy selection per week and cumulative reward over the backtest

---

## Citation & Acknowledgements

The code for this project was written independently by the author. Generative AI tools (Claude, Anthropic) were used to assist with code structure design, debugging, and documentation drafting. Full disclosure of AI tool usage is provided in Appendix B of the accompanying report.
```
