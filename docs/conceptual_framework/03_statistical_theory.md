# Statistics Theory: CapitalQuant

**How do we measure outperformance? Why XGBoost? What validation tests?**

---

## Core Statistical Problem

We need to predict which companies will **outperform the S&P 500** over the next 252 trading days (approximately 1 year). This is a **binary classification** problem where:

```
Target = 1 if Return_stock > Return_SP500
Target = 0 otherwise
```

---

## Why Binary Classification?

| Approach | Problem |
|----------|---------|
| Regression (predicting exact return) | Noisy, unstable, difficult to calibrate |
| Ranking (ordering companies) | Doesn't provide probability of outperformance |
| **Binary classification** | Clean target, probabilistic output, interpretable |

The binary framework aligns with the investment decision: "Will this stock beat the market?"

---

## The Model: XGBoost

### Why XGBoost?

| Feature | Benefit |
|---------|---------|
| **Gradient boosting** | Handles non-linear relationships automatically |
| **Regularization** | Prevents overfitting (L1/L2) |
| **Missing value handling** | Automatically learns best direction for missing values |
| **Tree-based** | No need for feature scaling |
| **Feature importance** | Built-in interpretability |
| **Speed** | Fast training and inference |
| **Walk-forward compatibility** | Easy to retrain on rolling windows |

### Mathematical Formulation

XGBoost minimizes the following objective:

```
L(θ) = Σ l(yi, ŷi) + Σ Ω(fk)
```

Where:
- `l(yi, ŷi)` = Logistic loss for binary classification
- `Ω(fk)` = Regularization term: `γT + ½λ||w||²`

The final prediction is a **probability**:
```
P(y=1 | x) = 1 / (1 + exp(-Σ fk(x)))
```

---

## The Target Variable

### Construction

For each company, for each day `t`:

```
Return_stock(t, t+252) = (Price_t+252 / Price_t) - 1
Return_SP500(t, t+252) = (SP500_t+252 / SP500_t) - 1

Target(t) = 1 if Return_stock > Return_SP500
Target(t) = 0 otherwise
```

### Why 252 trading days?

- **1-year horizon** matches typical investment timeframes
- **252 trading days** = 1 year of US market data
- **Long enough** for fundamentals to matter
- **Short enough** to be actionable

---

## Feature Engineering

### Feature Categories

| Category | Examples | Count |
|----------|----------|-------|
| **Fundamental** | ROE, ROA, Gross Margin, Net Margin | 15+ |
| **Growth** | Revenue Growth (1Y, 3Y, 5Y), EPS Growth | 10+ |
| **Valuation** | P/E, P/B, P/S, EV/EBITDA | 8+ |
| **Risk** | Beta, Volatility, Net Debt/EBITDA | 6+ |
| **Quality** | Current Ratio, Quick Ratio, Interest Coverage | 5+ |
| **Market** | Momentum, Size, Value factors | 6+ |

### Rolling Windows

Each feature is calculated over rolling windows to capture dynamic behavior:

```
Feature_t = RollingMetric(Data[t-window:t])
```

Common windows: 1Y, 3Y, 5Y, TTM (Trailing Twelve Months)

### Feature Normalization

For tree-based models (XGBoost), normalization is **not required** but we standardize for interpretability:

```
Z-score = (X - μ) / σ
```

---

## Walk-Forward Validation

### Why Walk-Forward?

Standard cross-validation **leaks future information** in time-series data. Walk-forward validation prevents this by respecting chronological order.

### The Process

```
Window 1: Train [2010-2014] → Test [2015]
Window 2: Train [2011-2015] → Test [2016]
Window 3: Train [2012-2016] → Test [2017]
...
Window N: Train [2018-2022] → Test [2023]
```

### Parameters

- **Training window:** 252 days × 4 years = ~1,008 trading days
- **Testing window:** 21 trading days (1 month)
- **Stride:** 21 trading days (monthly rebalancing)

### Why 252/21?

- **252 days (1 year):** Provides enough data for robust training
- **21 days (1 month):** Simulates monthly rebalancing cycle
- **Monthly stride:** Matches typical portfolio management frequency

---

## Evaluation Metrics

### Primary Metrics

| Metric | Definition | Why It Matters |
|--------|------------|----------------|
| **AUC-ROC** | Area under the ROC curve | Measures ranking ability (most important) |
| **Precision** | TP / (TP + FP) | Measures reliability of positive predictions |
| **Recall** | TP / (TP + FN) | Measures ability to find winners |
| **Accuracy** | (TP + TN) / Total | Overall correctness |
| **F1-Score** | 2 × (P × R) / (P + R) | Harmonic mean of precision and recall |

### Why AUC-ROC is Critical

The ROC curve measures the model's ability to **rank** companies correctly—which is exactly what investors need: a list of opportunities ordered by probability of outperformance.

```
AUC = 0.5 → Random (useless)
AUC = 0.7 → Good
AUC = 0.8 → Excellent
AUC = 0.9 → Outstanding
```

### Calibration

Probability outputs must be **well-calibrated**:

```
P(prediction) ≈ Actual frequency
```

**Calibration Curve:** Groups predictions into bins (e.g., 0-10%, 10-20%, ...) and compares mean predicted probability to actual outcome rate.

---

## SHAP for Explainability

### What is SHAP?

**SHAP (SHapley Additive exPlanations)** uses game theory to allocate contributions to each feature.

### The Equation

For each prediction, SHAP decomposes:

```
f(x) = φ₀ + Σ φᵢ
```

Where:
- `f(x)` = Prediction (relative to baseline)
- `φ₀` = Baseline prediction (average over training data)
- `φᵢ` = Contribution of feature `i`

### Properties

| Property | Description |
|----------|-------------|
| **Efficiency** | Contributions sum to the prediction |
| **Symmetry** | Equal features get equal contributions |
| **Dummy** | Uninformative features get zero contribution |
| **Additivity** | Can compute global importance from local SHAP |

### Global SHAP

Feature importance averaged across all predictions:

```
Feature Importanceᵢ = Mean(|φᵢ|) over all predictions
```

### Local SHAP

Per-prediction breakdown showing which features drove a specific score:

```
Score = Baseline + ROE_contribution + Revenue_contribution + ... + P/E_contribution
```

---

## Statistical Tests

### Validation Tests

| Test | What It Tests | Pass Criteria |
|------|---------------|---------------|
| **Calibration Brier Score** | Probability calibration | < 0.25 |
| **Hosmer-Lemeshow** | Goodness of fit | p > 0.05 |
| **Kolmogorov-Smirnov** | Distribution of predictions vs. actuals | p > 0.05 |

### Stability Tests

| Test | What It Tests | Pass Criteria |
|------|---------------|---------------|
| **SHAP Stability** | Feature importance consistent across windows | Correlation > 0.8 |
| **AUC Stability** | Performance consistent across time | Std dev < 0.05 |

---

## Risk of Overfitting

### Data Leakage Prevention

| Leakage Type | Prevention |
|--------------|------------|
| **Look-ahead bias** | Walk-forward validation |
| **Survivorship bias** | Include delisted companies |
| **Feature leakage** | Rolling windows only use past data |
| **Test contamination** | No test data used in training |

### Regularization

XGBoost hyperparameters to prevent overfitting:

| Parameter | Effect |
|-----------|--------|
| `max_depth` | Limits tree complexity (3-6 recommended) |
| `learning_rate` | Slows learning (0.01-0.1) |
| `subsample` | Uses random subset of data per tree (0.7-0.9) |
| `colsample_bytree` | Uses random subset of features per tree (0.7-0.9) |
| `reg_alpha` | L1 regularization (0-1) |
| `reg_lambda` | L2 regularization (1-5) |

---

## Sample Size Considerations

### Data Requirements

- **Companies:** 500 (S&P 500) × 15 years × 252 days = ~1.9M observations
- **Features:** ~50 per observation
- **Training window:** 1 year × 500 companies = 126,000 observations
- **Test window:** 1 month × 500 companies = 10,500 observations

### Minimum Observations per Feature

Rule of thumb: **10-30 observations per feature**

```
126,000 / 50 = 2,520 observations per feature → Excellent
```

---

## Comparison to Alternative Models

| Model | Pros | Cons | Verdict |
|-------|------|------|---------|
| **Logistic Regression** | Interpretable | Linear only | ❌ Too simplistic |
| **Random Forest** | Good ensemble | Harder to tune | ⚠️ Less accurate than XGBoost |
| **Neural Network** | Highly flexible | Black box, data hungry | ❌ Not explainable |
| **XGBoost** | Accurate, interpretable, fast | Requires tuning | ✅ Winner |
| **LSTM** | Handles sequences | Overkill, not interpretable | ❌ Not necessary |

---

## Summary: Statistical Foundation

| Component | Choice | Rationale |
|-----------|--------|-----------|
| **Target** | Binary outperformance | Clean, interpretable, actionable |
| **Model** | XGBoost | Accuracy, speed, interpretability |
| **Validation** | Walk-forward | Realistic, no leakage |
| **Evaluation** | AUC-ROC | Measures ranking ability |
| **Explainability** | SHAP | Game-theoretic, additivity |
| **Feature set** | 50+ fundamentals + market | Comprehensive coverage |

---

## Success Criteria for MVP

| Metric | Threshold |
|--------|-----------|
| **AUC-ROC** | > 0.70 |
| **Precision** | > 0.60 |
| **Recall** | > 0.55 |
| **Calibration** | Brier score < 0.25 |
| **SHAP Stability** | Feature rank correlation > 0.80 |

---

**CapitalQuant: Statistical rigor meets investment intelligence.**