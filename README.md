# SEC Filing Text Alpha

This repository presents a research pipeline for constructing an event-driven
equity alpha signal from unstructured SEC filings (10-K, 10-Q, and 8-K).

Using SEC EDGAR filings as a single standardized data source, the project
transforms textual characteristics—sentiment, uncertainty, complexity, novelty,
structure, timing, and event tags—into investable features. These features feed
LightGBM models that predict forward excess returns, which are converted into a
daily, dollar-neutral Alpha Score.

The emphasis is on economic interpretability, robustness, and realistic
backtesting rather than maximizing headline performance.

---

## Overview

**Data**
- Universe: S&P 500 constituents
- Source: SEC EDGAR filings (MD&A, Risk Factors, all 8-K items)
- Price data used only for labeling and backtesting

**Models**
- 10-K / 10-Q model: 63-day forward excess returns
- 8-K model: 10-day forward excess returns
- Learner: LightGBM with purged and embargoed time-series cross-validation

**Signal Construction**
- Event-level predictions are carried forward until expiration or next event
- Daily cross-sectional z-scoring and rank scaling
- Final Alpha Score ∈ [-100, +100], re-centered daily to enforce dollar neutrality

**Portfolio**
- Event-rebalanced long/short strategy
- Top 20% / bottom 20% by daily Alpha Score
- Tranche-based construction with turnover and transaction costs

---

## Repository Structure

notebooks/
01_feature_maker.ipynb # Text feature engineering
02_score_maker.ipynb # Labeling, modeling, daily alpha construction
03_backtest.ipynb # Portfolio construction and evaluation

data/
raw/ # Not included (user-supplied)
processed/ # Engineered features (generated)

output/
daily_alpha/ # Daily alpha matrices (generated)

cache/
finbert/ # Cached NLP model outputs


---

## Reproducibility Notes

- Raw SEC filing text and price data are **not included**.
- Notebooks document expected schemas and paths.
- The project is intended for research illustration rather than turnkey execution.

---

## Results (2020–2025)

- Net annualized return: ~0.9%
- Annualized volatility: ~6.8%
- Sharpe (net): ~0.14
- Performance is thin but persistent and regime-dependent, consistent with
  prior literature on textual and disclosure-based signals.

---

## Key Takeaways

- Unstructured disclosure text contains modest but exploitable information.
- Event-driven signals (especially 8-Ks) dominate persistent effects.
- Interpretability (via SHAP) is critical for validating textual alphas.
- Portfolio construction and execution assumptions materially affect outcomes.

---

## Disclaimer

This repository is for research and educational purposes only.
It does not constitute investment advice or a production trading system.
