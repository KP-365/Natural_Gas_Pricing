# Weather-Driven Natural Gas Demand Forecasting with Calibrated Uncertainty

**ECS7036P — Applications of AI, Coursework 2 (Individual Project)**
**Kayleb Parkes | 251129879**

## Overview

This project builds and rigorously tests calibrated uncertainty for weekly U.S. natural gas storage forecasts, comparing a point-estimate baseline, a Bayesian linear regression, and a heteroscedastic neural network, and tests whether that uncertainty carries genuine predictive value for realised natural gas price volatility. Full methodology, results, and discussion are in the accompanying report.

## Contents

- `storage_volatility_final_analysis.ipynb` — the complete Jupyter notebook implementing the full 8-stage pipeline (data ingestion, three forecasting models, walk-forward backtest, volatility relevance test, robustness checks, Thompson sampling decision layer, and the practical deliverable models).
- `Parkes_ECS7036P_Report.tex` — the LaTeX source for the written report.
- `README.md` — this file.

No large source data or library files are included in this submission, per the submission instructions. All data is fetched live from the free public APIs listed below when the notebook is run.

## Dataset Links

All data used in this project is free, public, and fetched programmatically at runtime, no data files are bundled with this submission.

- **EIA Weekly Natural Gas Storage Report**: https://www.eia.gov/naturalgas/storage/dashboard/ (accessed via the EIA v2 API, `natural-gas/stor/wkly` route)
- **EIA Natural Gas Futures Prices** (Henry Hub spot): accessed via the EIA v2 API, `natural-gas/pri/fut` route
- **EIA Underground Natural Gas Storage Capacity**: accessed via the EIA v2 API, `natural-gas/stor/cap` route
- **NOAA Climate Prediction Center, population-weighted national HDD/CDD**: https://www.ncdc.noaa.gov/cdo-web/ (free plain-text files, no API key required)

## Requirements

- Python 3.12
- An EIA API key (a working key is already included in the notebook's configuration cell for marking convenience, free to obtain your own at https://www.eia.gov/opendata/register.php if needed)
- Key libraries: NumPy, pandas, Requests, PyMC 6.2, PyTorch 2.13, statsmodels, scikit-learn, SciPy, Matplotlib

Install any missing packages with:
```bash
pip install pymc torch statsmodels scikit-learn scipy matplotlib pandas numpy requests
```

## How to Run

1. Open `storage_volatility_final_analysis.ipynb` in Google Colab (or a local Jupyter environment).
2. The EIA API key is already set in the configuration cell near the top of the notebook, no setup is required to run it as submitted. If the key stops working or you want to use your own, replace it there:
   ```python
   EIA_API_KEY = "your-key-here"
   ```
3. Run all cells from top to bottom (`Runtime → Run all` in Colab). No GPU is required; the full pipeline, including the walk-forward backtest, completes in a few minutes on standard Colab compute.
4. All results (tables, figures, and printed statistics referenced in the report) are generated as the notebook runs, no separate data files or pre-computed results are needed.

## Notes

- An EIA API key is included directly in the notebook so the pipeline can be run and marked without any additional setup. EIA keys are free and carry no sensitive access, this is provided purely for reproducibility.
- All reported results in the accompanying report reflect actual notebook execution output on real data, not synthetic or estimated figures.
- Full disclosure of AI tool usage for this project is provided in Appendix B of the report.
