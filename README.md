# S&P 500 Credit Health Analysis

**Which S&P 500 companies are financially deteriorating - and can a data-driven scoring model identify them systematically?**

This project builds an automated credit health scoring pipeline for 389 S&P 500 companies across 9 sectors, covering fiscal years 2022-2025. It mirrors the type of quantitative credit analysis used in fixed income research and investment decision support.

---

## Key Findings

| Finding | Value |
|---|---|
| Companies analyzed | 389 across 9 sectors |
| Fiscal years covered | 2022-2025 |
| Companies rated AAA or AA | 36 (9.3%) |
| Companies on risk watchlist | 76 |
| Sectors improving 2022-2025 | Communication Services, Information Technology |
| Sectors declining 2022-2025 | Consumer Staples, Consumer Discretionary |

**Risk signals that align with market reality:**
- SBUX (score 22.6): 3 consecutive years of decline, same-store sales dropped significantly 2023-2025
- UNH (score 24.3): rising medical loss ratios and regulatory pressure throughout 2024-2025
- F (score 18.6): EV transition losses and sustained margin compression

---

## Project Structure

```
sp500-credit-analysis/
├── sp500_credit_analysis.ipynb   - Full pipeline: ETL, scoring, visualization
├── data/
│   ├── sp500_raw.csv             - Raw extracted financial data (389 companies x 4 years)
│   ├── sp500_clean.csv           - Cleaned data with all 6 metrics calculated
│   └── sp500_scores.csv          - Final credit scores and letter ratings
├── outputs/
│   ├── 01_rating_distribution.png
│   ├── 02_sector_scores.png
│   ├── 03_score_trend.png
│   ├── 04_company_scatter.png
│   └── 05_risk_watchlist.png
└── README.md
```

---

## Data Sources

| Source | What | Access |
|---|---|---|
| yfinance | Income statement, balance sheet, cash flow (2022-2025) | Free Python library |
| GitHub (datasets/s-and-p-500-companies) | S&P 500 constituent list with GICS sectors | Public CSV |

---

## Methodology

### 1. Extract

- Pulled 4 fiscal years of financial statements for 396 companies via yfinance API
- For each company: income statement (revenue, EBITDA, EBIT, interest expense), balance sheet (current assets/liabilities, total liabilities, equity), and cash flow (free cash flow)
- Excluded Financials and Real Estate sectors: standard credit metrics do not apply to banks (interest income is revenue) or REITs (use FFO not FCF)

### 2. Transform - ETL Pipeline

**Interest expense imputation**: yfinance has data gaps for recent fiscal years on some companies. We use CAGR-based forward projection when 3+ years of history are available, and forward fill otherwise. CAGR is capped at +-30% to prevent outlier distortion.

**Outlier handling (Winsorization)**:
- Debt-to-Equity capped at 99th percentile - extreme leverage ratios add no additional signal
- Interest Coverage capped between -20 and +50 - beyond these thresholds all companies fall in the same risk bucket
- Revenue Growth capped between -50% and +100% - controls for one-time events

**Negative equity**: 109 companies have negative stockholders equity (due to aggressive stock buybacks). D/E ratio is undefined in this case - these companies are assigned the maximum risk D/E value.

### 3. Six Credit Health Metrics

| Metric | Formula | Weight | Direction |
|---|---|---|---|
| EBITDA Margin | EBITDA / Revenue | 20% | Higher is better |
| Debt-to-Equity | Total Liabilities / Equity | 20% | Lower is better |
| Current Ratio | Current Assets / Current Liabilities | 15% | Higher is better |
| Interest Coverage | EBIT / Interest Expense | 20% | Higher is better |
| FCF Margin | Free Cash Flow / Revenue | 15% | Higher is better |
| Revenue Growth YoY | (Rev_t - Rev_t-1) / Rev_t-1 | 10% | Higher is better |

### 4. Scoring Model

Each metric is ranked by **within-sector percentile** to control for structural industry differences. A utility company carrying high debt is normal for that industry - comparing it directly to a tech company would be misleading.

- Percentile scores are scaled to 0-100 and weighted to produce a final credit score
- Ratings: AAA (85+), AA (75-85), A (65-75), BBB (55-65), BB (45-55), B (35-45), CCC (below 35)

### 5. Risk Detection

Companies flagged if credit score declined for 2+ consecutive years. 76 companies identified on the watchlist as of latest fiscal year.

---

## Limitations and Future Work

**Current limitations:**
- Weights are judgment-based, informed by credit rating agency methodology but not statistically derived from historical default data
- Interest expense imputation assumes stable debt structure - may be inaccurate for companies undergoing major refinancing
- Negative equity companies may be over-penalized: OTIS for example has strong cash flows despite negative book equity from buybacks
- Model is correlational - a low score indicates financial stress but does not estimate default probability

**Future improvements:**
- Use logistic regression on historical default data (Moody's default database) to derive statistically optimal weights
- Add sector-specific metrics (same-store sales for retail, reserve ratios for insurance)
- Validate model signals against CDS spreads as an external benchmark
- Build an interactive Plotly dashboard for company-level drill-down

---

## How to Run

```bash
pip install yfinance pandas numpy matplotlib requests

# First run: fetches data from yfinance (~6-7 minutes), caches results
# Subsequent runs: loads from cache, completes in seconds
jupyter notebook sp500_credit_analysis.ipynb
```

---

## Skills Demonstrated

`Python` `yfinance API` `ETL pipeline` `Pandas` `NumPy` `Data cleaning` `Winsorization` `CAGR imputation` `Percentile ranking` `Weighted scoring model` `Matplotlib` `Financial analysis` `Credit risk` `S&P 500`
