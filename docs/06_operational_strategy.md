# CapitalQuant Operational Strategy

**How the platform operates to produce explainable investment intelligence**

---

## Objective

CapitalQuant monitors the financial performance of S&P 500 companies across multiple dimensions to estimate the probability that each company will outperform the market. Rather than providing a single black-box prediction, the platform combines machine learning, explainability, and documentary evidence to produce transparent, auditable investment recommendations.

The objective is to surface investment opportunities where fundamentals, market behavior, and documentary evidence align to support a clear investment thesis.

The output of all operational levels feeds into the **Investment Score** — a calibrated probability that a company will outperform the S&P 500 over the next 252 trading days. The score is intended for ranking opportunities rather than estimating expected returns. Each score is accompanied by SHAP explanations and RAG-evidence citations.

---

## Level 1: Market Data Acquisition

**Purpose:** Establish the price and return baseline for all companies in the investment universe.

CapitalQuant continuously acquires and processes market data to calculate returns, volatility, momentum, and risk metrics.

### Monitored Data

| Data | Source | Frequency |
|------|--------|-----------|
| Daily OHLCV prices | Alpha Vantage / Stooq | Daily after market close |
| Structured financial statements | SEC XBRL API | As filed (quarterly) |
| 10-K and 10-Q filings (text for RAG) | SEC EDGAR | As filed |
| Macroeconomic indicators | FRED API | Daily |

### Key Calculations

| Metric | Calculation | Purpose |
|--------|-------------|---------|
| **Total Return** | (Price_t / Price_{t-252}) - 1 | Outperformance target |
| **Volatility** | Standard deviation of daily returns | Risk adjustment |
| **Momentum** | Price change over 1M, 3M, 6M, 12M | Trend detection |
| **Beta** | Covariance with S&P 500 / Variance | Market sensitivity |
| **Max Drawdown** | Maximum peak-to-trough decline | Risk assessment |

### What This Level Provides

- Complete price history for all S&P 500 constituents
- Company returns aligned with S&P 500 for target calculation
- Volatility and risk metrics for SHAP explanations
- Historical data for walk-forward validation

**Output to Investment Score:** Price-based features (momentum, volatility, beta, drawdown) used in XGBoost prediction.

---

## Level 2: Fundamental Intelligence

**Purpose:** Quantify corporate financial health, growth, and valuation using standardized, machine-readable data.

CapitalQuant uses **SEC XBRL API** to extract structured financial data directly from official filings, eliminating manual parsing errors and ensuring point-in-time correctness.

### XBRL Concepts Extracted

| Category | Concepts | Derived Metrics |
|----------|----------|-----------------|
| **Income Statement** | Revenue, Net Income, Gross Profit, Operating Income | Growth rates, margins |
| **Balance Sheet** | Assets, Liabilities, Equity, Cash, Debt | Leverage, liquidity |
| **Cash Flow** | Operating Cash Flow, Capital Expenditures, Free Cash Flow | Cash flow quality |
| **Valuation** | Shares Outstanding, EPS | P/E, P/B, P/S |

### Fundamental Features Calculated

| Feature | Calculation | Window |
|---------|-------------|--------|
| **Revenue Growth** | (Revenue_t - Revenue_{t-1}) / Revenue_{t-1} | 1Y, 3Y, 5Y |
| **EPS Growth** | (EPS_t - EPS_{t-1}) / EPS_{t-1} | 1Y, 3Y, 5Y |
| **ROE** | Net Income / Shareholders' Equity | TTM |
| **ROA** | Net Income / Total Assets | TTM |
| **Gross Margin** | Gross Profit / Revenue | TTM |
| **Net Margin** | Net Income / Revenue | TTM |
| **Debt/Equity** | Total Debt / Shareholders' Equity | Current |
| **Current Ratio** | Current Assets / Current Liabilities | Current |
| **Free Cash Flow Margin** | Free Cash Flow / Revenue | TTM |
| **Interest Coverage** | EBIT / Interest Expense | TTM |

### Point-in-Time Guarantee

All fundamental data is stored with:

- **valid_at:** The period the data describes (e.g., Q4 2023)
- **known_at:** When the filing was published

This ensures no look-ahead bias in training or inference.

### What This Level Provides

- 50+ fundamental features for XGBoost
- Feature contributions for SHAP explanations
- Evidence for RAG (directly traceable to XBRL concepts)
- Point-in-time correct training data

**Output to Investment Score:** Fundamental features (growth, profitability, leverage, liquidity, valuation) used in XGBoost prediction.

---

## Level 3: Macroeconomic Context

**Purpose:** Provide the economic environment context that influences all companies simultaneously.

CapitalQuant uses **FRED API** to incorporate macroeconomic indicators that affect earnings, valuation multiples, and risk premiums.

### Monitored Indicators

| Indicator | What It Signals | Impact on Companies |
|-----------|-----------------|---------------------|
| **Fed Funds Rate** | Monetary policy stance | Discount rate, borrowing costs |
| **10-Year Treasury Yield** | Risk-free rate | Valuation multiples |
| **CPI Inflation** | Price pressure | Real earnings, margins |
| **GDP Growth** | Economic cycle | Revenue growth expectations |
| **Unemployment Rate** | Consumer health | Credit quality, spending |
| **Industrial Production** | Manufacturing activity | Cyclical sector earnings |
| **Consumer Sentiment** | Confidence | Retail, consumer spending |

### What This Level Provides

- Macro context for SHAP explanations
- Risk regime indicator (risk-on / risk-off)
- Sector-level impact adjustment
- Baseline for expected returns

**Output to Investment Score:** Macro features (interest rates, inflation, GDP growth) used in XGBoost prediction.

---

## Level 4: Data Validation & Quality

**Purpose:** Ensure data integrity before it enters the feature store.

All data passes through validation gates before becoming eligible for model training or inference.

### Validation Checks

| Check | What It Detects | Action on Failure |
|-------|-----------------|-------------------|
| **Schema Validation** | Missing columns, wrong types | Reject batch |
| **Range Checks** | Negative prices, extreme outliers | Reject row, alert |
| **Completeness** | Missing expected records | Alert, use fallback |
| **Freshness** | Stale data | Alert, escalate |
| **Duplicate Detection** | Duplicate filings | Deduplicate, log |
| **Ticker Changes** | Historical ticker mapping | Apply mapping, log |
| **Corporate Actions** | Splits, dividends | Apply adjustment factor |
| **Restatements** | Amended filings | Update known_at, preserve history |

### Data Quality Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DATA QUALITY PIPELINE                            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  Raw Data Ingested                                                   │
│        │                                                             │
│        ▼                                                             │
│  Schema Validation                                                   │
│        │                                                             │
│        ▼                                                             │
│  Range & Business Logic Checks                                      │
│        │                                                             │
│        ▼                                                             │
│  Completeness & Freshness Checks                                    │
│        │                                                             │
│  ┌─────┴─────┐                                                      │
│  │           │                                                      │
│  ▼           ▼                                                      │
│ Pass      Fail → Quarantine → Alert → Manual Review                │
│  │                                                                  │
│  ▼                                                                  │
│  Silver Zone (Validated Data)                                       │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

### What This Level Provides

- Confidence that training data is correct
- Protection against data corruption
- Audit trail for data quality issues
- Early warning of data source problems

**Output to Feature Store:** Validated, clean data ready for feature engineering.

---

## Level 5: Feature Engineering & Store

**Purpose:** Transform raw data into a consistent, versioned feature set for model training and inference.

All features from Levels 1-3 are computed, validated, and stored in a **Feature Store** with offline and online components.

### Feature Categories

| Category | Count | Examples |
|----------|-------|----------|
| **Market** | 15 | Momentum, volatility, beta, max drawdown |
| **Fundamental** | 20 | ROE, revenue growth, margins, leverage |
| **Valuation** | 8 | P/E, P/B, P/S, EV/EBITDA |
| **Macro** | 5 | Fed Funds, Treasury yield, CPI, GDP |
| **Quality** | 5 | Current ratio, quick ratio, interest coverage |
| **Total** | **53** | |

### Feature Store Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    FEATURE STORE ARCHITECTURE                       │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │                    OFFLINE STORE                                │ │
│  │  • 15 years of historical features                            │ │
│  │  • Point-in-time and bitemporal                               │ │
│  │  • Used for walk-forward training                             │ │
│  │  • Feature versioning                                          │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    ONLINE STORE                                │ │
│  │  • Latest features for all companies                          │ │
│  │  • Low-latency inference                                      │ │
│  │  • TTL: 24 hours                                              │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

### What This Level Provides

- Reproducible training data
- Low-latency inference features
- Complete feature lineage
- Versioning for A/B testing

**Output to Investment Score:** Feature vectors for training and inference.

---

## Level 6: Machine Learning & Explainability

**Purpose:** Estimate the probability of outperformance and explain why each company receives its score.

This is the core of CapitalQuant's predictive and explainability engine.

### Model Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    ML + SHAP ARCHITECTURE                           │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │                    XGBOOST CLASSIFIER                           │ │
│  │                                                                  │ │
│  │  Target: Outperform S&P 500 over next 252 trading days         │ │
│  │  Objective: binary:logistic                                    │ │
│  │  Model Class: Gradient Boosting (tree-based ensemble)          │ │
│  │  Rationale: Handles non-linear relationships, missing         │ │
│  │             values, and mixed data types without scaling      │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    SHAP EXPLAINER                              │ │
│  │  • Game-theoretic feature attribution                         │ │
│  │  • Global SHAP: Feature importance across all predictions     │ │
│  │  • Local SHAP: Per-company contribution breakdown             │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

### Prediction

For each company, the model outputs:

```
P(Outperform) = XGBoost(features)  # Calibrated probability between 0 and 1

Investment Score = P(Outperform) × 100
```

The Investment Score is a calibrated probability of outperforming the S&P 500 over the next 252 trading days. It is intended for ranking opportunities rather than estimating expected returns.

### Explanation

For each prediction, SHAP decomposes:

```
Investment Score = Baseline + Σ Feature_Contribution_i

Where:
- Baseline: Average score across training data
- Feature_Contribution_i: Contribution of feature i (positive or negative)
```

### Walk-Forward Validation

The model is validated using walk-forward validation to prevent look-ahead bias:

```
Window 1: Train [2010-2014] → Test [2015]
Window 2: Train [2011-2015] → Test [2016]
Window 3: Train [2012-2016] → Test [2017]
...
Window N: Train [2018-2022] → Test [2023]
```

**Validation Metrics:**

| Metric | Target | What It Measures |
|--------|--------|------------------|
| **AUC-ROC** | > 0.55 | Ranking ability |
| **Precision@k** | > 0.55 | Top-decile performance |
| **Calibration** | Brier < 0.25 | Probability accuracy |
| **Stability** | Std dev < 0.05 | Consistency across windows |

### What This Level Provides

- Ranked list of 500 companies by Investment Score
- SHAP breakdown for every prediction
- Feature importance (global and local)
- Calibrated probabilities
- Walk-forward validation results

**Output to Investment Decision:** Investment Score + SHAP explanation + supporting features.

---

## Level 7: Document Intelligence (RAG)

**Purpose:** Provide documentary evidence from SEC filings to support every recommendation.

This level transforms unstructured SEC filings into **verifiable, source-cited evidence** that analysts can use to validate the model's recommendations.

**Important:** The RAG pipeline retrieves evidence **after** the prediction is made. It does not influence the Investment Score. Its purpose is to support analyst verification, not to contribute to the model's output.

### RAG Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    RAG ARCHITECTURE                                 │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │                    DOCUMENT INGESTION                            │ │
│  │  • SEC EDGAR 10-K and 10-Q filings                             │ │
│  │  • Unstructured.io for text and table extraction              │ │
│  │  • Metadata: ticker, filing_type, year, section, page         │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    CHUNKING & EMBEDDING                        │ │
│  │  • Text chunks: 1,024 tokens, 128 overlap                    │ │
│  │  • Table chunks: Preserved structure                          │ │
│  │  • Embedding: all-MiniLM-L6-v2 (local, open-source)           │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    VECTOR STORE                                │ │
│  │  • Hybrid search (semantic + keyword)                         │ │
│  │  • Metadata filters                                           │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    RETRIEVAL & RERANKING                       │ │
│  │  1. Semantic search on user query or ticker context          │ │
│  │  2. Cross-encoder reranking                                   │ │
│  │  3. Return top-5 chunks with source citations                │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

### What This Level Provides

- Natural language Q&A over SEC filings
- Source citations (section, page, filing date)
- Table extraction for financial statements
- Evidence to verify model recommendations

**Output to Investment Decision:** Document evidence + citations for every recommendation.

---

## Level 8: Investment Decision Synthesis

**Purpose:** Combine prediction, explanation, and evidence into a transparent investment recommendation.

This level synthesizes outputs from all previous levels into a single, actionable recommendation with complete auditability.

### Decision Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│                    INVESTMENT DECISION FLOW                         │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │                    COMPANY SCREENING                            │ │
│  │                                                                  │ │
│  │  All S&P 500 companies sorted by Investment Score              │ │
│  │  Top 20% = High Conviction                                      │ │
│  │  Middle 60% = Neutral                                           │ │
│  │  Bottom 20% = Low Conviction                                    │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    SHAP EXPLANATION                            │ │
│  │                                                                  │ │
│  │  For each high-conviction company:                             │ │
│  │  • Baseline score (average)                                    │ │
│  │  • Top 5 positive contributors                                │ │
│  │  • Top 5 negative contributors                                 │ │
│  │  • Net score                                                   │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    DOCUMENTARY EVIDENCE                        │ │
│  │                                                                  │ │
│  │  For each high-conviction company:                             │ │
│  │  • "What does the latest 10-K say about growth?"              │ │
│  │  • Retrieved evidence with citations                          │ │
│  │  • Supporting financial statement tables                      │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    RECOMMENDATION                              │ │
│  │                                                                  │ │
│  │  Analyst receives:                                             │ │
│  │  1. Investment Score                                           │ │
│  │  2. SHAP breakdown (what drove the score)                     │ │
│  │  3. Documentary evidence (supporting citations)               │ │
│  │  4. Walk-forward validation results                           │ │
│  │                                                                  │ │
│  │  Analyst retains final judgment.                               │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

### Human Feedback Loop

All analyst decisions are recorded for future evaluation:

```
Analyst Accepts/Rejects Recommendation
        │
        ▼
Decision Recorded (audit trail)
        │
        ▼
Periodic Review: Accepted vs. Actual Outperformance
        │
        ▼
Performance Attribution: Where did the model succeed or fail?
```

### What This Level Provides

- Ranked investment opportunities
- Explanations for why each company scored highly
- Verifiable evidence from SEC filings
- Complete audit trail
- Human feedback loop for performance evaluation

**Output to Analyst:** Explainable investment recommendation with full transparency.

---

## Level 9: Monitoring & Model Lifecycle

**Purpose:** Ensure the model remains performant, explainable, and reliable over time.

This level implements continuous monitoring, drift detection, and automated retraining.

### Monitoring Metrics

| Metric | What It Measures | Alert Threshold |
|--------|------------------|-----------------|
| **Investment Score Drift** | Distribution change in scores | KS p < 0.05 |
| **Feature Drift** | Distribution change in features | KS p < 0.05 |
| **Model Performance** | Rolling AUC-ROC | Drop > 5% from baseline |
| **Prediction Latency** | p95 response time | > 500ms |
| **API Availability** | Uptime percentage | < 99.9% |
| **SHAP Consistency** | Feature importance correlation | Correlation < 0.8 |

### MLOps Lifecycle

```
┌──────────────────────────────────────────────────────────────────────┐
│                    MLOPS LIFECYCLE                                   │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  DATA INGESTION                                                      │
│  ────────────────────────────────────────────────────────────────── │
│  • Alpha Vantage, SEC XBRL, FRED, SEC EDGAR                       │
│  • Great Expectations validation                                   │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  FEATURE ENGINEERING                                                 │
│  ────────────────────────────────────────────────────────────────── │
│  • 53 features computed                                            │
│  • Point-in-time + bitemporal                                     │
│  • Feature Store updated                                           │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  MODEL TRAINING                                                      │
│  ────────────────────────────────────────────────────────────────── │
│  • XGBoost with walk-forward validation                           │
│  • SHAP explanations computed                                      │
│  • Performance metrics tracked                                     │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  MODEL VALIDATION                                                    │
│  ────────────────────────────────────────────────────────────────── │
│  • Walk-forward test set                                           │
│  • AUC-ROC > 0.55 required                                         │
│  • SHAP stability checked                                          │
│  • Calibration verified                                            │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  DEPLOYMENT                                                          │
│  ────────────────────────────────────────────────────────────────── │
│  • Canary deployment                                                │
│  • Health checks                                                   │
│  • Rollback capability                                             │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  MONITORING                                                          │
│  ────────────────────────────────────────────────────────────────── │
│  • Feature drift (KS test)                                         │
│  • Concept drift (rolling AUC)                                     │
│  • SHAP drift (distribution shift)                                │
│  • Latency, error rate, throughput                                │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  RETRAINING TRIGGER                                                  │
│  ────────────────────────────────────────────────────────────────── │
│  • Drift detected (KS p < 0.05)                                   │
│  • Monthly scheduled                                               │
│  • Performance drop > 5%                                          │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  MODEL RETRAINING                                                    │
│  ────────────────────────────────────────────────────────────────── │
│  • Walk-forward on 3 years of data                                 │
│  • New model registered                                            │
│  • Validation passed → Promote to production                      │
│  • Validation failed → Alert, retain previous                     │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

### What This Level Provides

- Continuous monitoring of model health
- Early detection of data and concept drift
- Automated retraining pipeline
- Model versioning and rollback
- Performance dashboards

**Output to Operations:** Alerts, dashboards, and automated retraining decisions.

---

## How the Levels Work Together

| Level | Domain | Primary Output | Feeds Into |
|-------|--------|----------------|------------|
| **1** | Market Data | Prices, returns, volatility | Investment Score (XGBoost) |
| **2** | Fundamentals | 50+ financial ratios, point-in-time | Investment Score + SHAP |
| **3** | Macro Context | Interest rates, inflation, GDP | Investment Score |
| **4** | Data Validation | Clean, validated data | Feature Store |
| **5** | Feature Store | Versioned feature vectors | Training + Inference |
| **6** | ML + SHAP | Investment Score + Explanations | Investment Decision |
| **7** | RAG | Document evidence + Citations | Investment Decision |
| **8** | Synthesis | Transparent recommendation | Analyst |
| **9** | Monitoring | Drift detection, retraining | Model Lifecycle |

---

## Summary: Operational Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│                    CAPITALQUANT OPERATIONAL FLOW                    │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  LEVEL 1: MARKET DATA                                                │
│  ────────────────────────────────────────────────────────────────── │
│  Alpha Vantage / Stooq → Prices, returns, volatility               │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  LEVEL 2: FUNDAMENTALS                                               │
│  ────────────────────────────────────────────────────────────────── │
│  SEC XBRL → 50+ ratios (ROE, growth, margins, leverage)           │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  LEVEL 3: MACRO CONTEXT                                              │
│  ────────────────────────────────────────────────────────────────── │
│  FRED API → Interest rates, inflation, GDP                        │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  LEVEL 4: DATA VALIDATION                                            │
│  ────────────────────────────────────────────────────────────────── │
│  Great Expectations → Schema, range, completeness, freshness      │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  LEVEL 5: FEATURE STORE                                              │
│  ────────────────────────────────────────────────────────────────── │
│  Offline + Online → Feature vectors for training and inference    │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  LEVEL 6: ML + SHAP                                                  │
│  ────────────────────────────────────────────────────────────────── │
│  XGBoost → Investment Score                                         │
│  SHAP → Explanation (positive/negative contributors)               │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  LEVEL 7: RAG                                                        │
│  ────────────────────────────────────────────────────────────────── │
│  SEC EDGAR → Document evidence + Source citations                  │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  LEVEL 8: SYNTHESIS                                                  │
│  ────────────────────────────────────────────────────────────────── │
│  Score + Explanation + Evidence → Transparent Recommendation      │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  ANALYST DECISION                                                    │
│  ────────────────────────────────────────────────────────────────── │
│  Human judgment with AI-supported evidence                         │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  LEVEL 9: MONITORING                                                 │
│  ────────────────────────────────────────────────────────────────── │
│  Drift detection → Retraining → Canary Deployment → Rollback      │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

*CapitalQuant does not attempt to replace investment analysts. It augments their decision-making process by combining reliable financial data, explainable machine learning, and documentary evidence into a transparent investment research workflow. Every recommendation remains reproducible, auditable, and grounded in verifiable data.*