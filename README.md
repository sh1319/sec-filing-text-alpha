# SEC Filing Text Alpha

A research pipeline that turns **unstructured SEC disclosure text** (10-K, 10-Q, 8-K) into an **event-driven equity alpha signal**. The project uses SEC EDGAR filings as a single, standardized data source; extracts textual features (sentiment, readability, novelty, structure, timing, event topics); trains LightGBM models to predict forward excess returns; and builds a daily, dollar-neutral Alpha Score for backtesting.

The design prioritizes **economic interpretability**, **robustness**, and **realistic backtesting** over headline performance.

---

## Why This Project?

- **Single-source signal**: All inputs come from SEC filings and market data—no alternative data or proprietary feeds.
- **Interpretability**: SHAP is used to explain which textual features drive predictions.
- **Event-driven**: Predictions are carried forward until the next filing or horizon end, then evaluated with transaction costs and turnover.
- **Production-oriented structure**: Clear separation of feature engineering, model training, and backtest, with consistent paths and config.

---

## Tech Stack

| Layer | Tools |
|-------|--------|
| **NLP / sentiment** | Transformers (FinBERT), regex (tokenization, sentence splitting) |
| **Features** | scikit-learn (TF-IDF), custom readability and topic logic |
| **Modeling** | LightGBM, purged/embargoed time-series CV |
| **Explainability** | SHAP (TreeExplainer, summary and dependence plots) |
| **Data** | pandas, NumPy; Parquet I/O |
| **Environment** | Python 3, Jupyter notebooks |

---

## Repository Structure

```text
.
├── alpha_feature_maker.ipynb   # 1. Text feature engineering (FinBERT, readability, novelty, 8-K topics)
├── alpha_score_maker.ipynb     # 2. Labels, LightGBM training, daily Alpha Score construction
├── alpha_tester.ipynb          # 3. Portfolio backtest, risk metrics, IC, plots
├── README.md
│
├── OUTPUT/                     # Generated (create by running notebooks)
│   ├── features_event_8k.parquet
│   ├── features_event_10kq.parquet
│   ├── alpha_scores_event_daily_wide.parquet
│   └── shap_*.png
│
├── price_data/                 # User-supplied: (date, ticker, px) adjusted prices
│   └── prices_3col_adjusted_2001_2025.parquet
│
├── final_view_cleaned.parquet  # User-supplied: cleaned SEC filing text (see schema below)
├── _cache/                     # FinBERT cache (optional, created at runtime)
└── _checkpoints/               # Optional checkpoints
```

**Expected input schema (cleaned filings):**  
Columns include `ticker`, `date`, `file_type` (e.g. `10-K`, `10-Q`, `8-K`), `content` (full text), and for 10-K/10-Q optionally `rf` (risk factors section). Paths and column names are configurable at the top of each notebook.

---

## How to Run

1. **Prepare data**
   - Place cleaned filing text in `final_view_cleaned.parquet` (or set `INPUT_CLEAN` in `alpha_feature_maker.ipynb`).
   - Place adjusted price data in `price_data/prices_3col_adjusted_2001_2025.parquet` (or set `PRICES_3COL` / `PRICE_PARQUET` in the score maker and tester).

2. **Run in order**
   - **alpha_feature_maker.ipynb** → writes `OUTPUT/features_event_8k.parquet` and `OUTPUT/features_event_10kq.parquet`.
   - **alpha_score_maker.ipynb** → reads those feature files and prices, trains models, writes `OUTPUT/alpha_scores_event_daily_wide.parquet` and SHAP plots.
   - **alpha_tester.ipynb** → reads the alpha matrix and prices, runs the backtest and risk/IC analysis.

3. **Optional**
   - Set `FINBERT_MODEL` or paths in the first cell of each notebook if you use different locations or models.

Raw SEC text and price data are **not included**; the repo is set up for research and illustration with your own data.

---

## Methodology (Summary)

| Aspect | Choice |
|--------|--------|
| **Universe** | S&P 500 (or subset with sufficient 10-K/10-Q history) |
| **Filing text** | MD&A, Risk Factors, full 8-K content |
| **10-K/10-Q target** | 63-day forward excess return (vs market) |
| **8-K target** | 10-day forward excess return |
| **Validation** | Purged and embargoed time-series CV |
| **Signal** | Event-level predictions carried to horizon or next event; daily cross-sectional z-score then rank scaled to [-100, 100]; re-centered for dollar neutrality |
| **Portfolio** | Long/short top/bottom 20% by Alpha Score; tranche construction with turnover and transaction costs |

---

## Results (2020–2025)

- **Net annualized return**: ~0.9%
- **Annualized volatility**: ~6.8%
- **Sharpe (net)**: ~0.14

Performance is modest but persistent and regime-dependent, in line with prior work on textual and disclosure-based signals.

---

## Takeaways

- Unstructured disclosure text contains **modest but exploitable** information for cross-sectional return prediction.
- **Event-driven signals** (especially 8-Ks) contribute meaningfully to the combined alpha.
- **Interpretability** (e.g. SHAP) is important for validating and trusting textual alphas.
- **Portfolio construction** and execution assumptions (turnover, costs, rebalance rules) materially affect outcomes.

---

## Disclaimer

This repository is for **research and educational purposes only**. It does not constitute investment advice or a production trading system.
