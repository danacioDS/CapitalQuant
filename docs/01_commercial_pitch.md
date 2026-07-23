# CapitalQuant Commercial Pitch

**AI-Powered Investment Decision Support Platform**

---

## One Sentence

**CapitalQuant combines machine learning, explainability, and documentary evidence to predict which companies will outperform the market — and show you exactly why.**

---

## The Problem

Investment professionals face three structural challenges that create demand for decision support tools:

| Challenge | Description |
|-----------|-------------|
| **Information overload** | Thousands of companies, hundreds of financial ratios, dozens of annual reports per company |
| **Cognitive biases** | Decisions influenced by market noise, emotions, or confirmation bias |
| **Lack of traceability** | Difficulty reconstructing and objectively explaining why a specific investment decision was made |

Traditional analysis is split into two incomplete halves:

- **Quantitative models** provide predictions but rarely explain *why* a company scored highly.
- **Fundamental research** provides depth but is time-consuming and inconsistent across analysts.

The true danger is not a lack of information. It is making decisions based on incomplete analysis, unconscious bias, or predictions that cannot be explained or audited.

---

## The Solution

CapitalQuant brings together three proven technologies into a single investment decision framework:

```
┌──────────────────────────────────────────────────────────────────────┐
│                    PREDICT. EXPLAIN. VERIFY.                        │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │                    PREDICT                                      │ │
│  │  XGBoost predicts probability of outperforming the S&P 500    │ │
│  │  over the next 252 trading days                               │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    EXPLAIN                                     │ │
│  │  SHAP decomposes every prediction into individual feature    │ │
│  │  contributions — showing exactly what drove the score         │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    VERIFY                                     │ │
│  │  RAG retrieves source-cited evidence from SEC filings to     │ │
│  │  support every recommendation                                │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

**"When a company scores highly, you don't just see the number. You see exactly what drove it — and can verify it against official documents."**

---

## Who Is It For?

| Segment | How They Benefit |
|---------|------------------|
| **Equity Research Analysts** | Save 5-10 hours per company analyzed; generate better-documented recommendations |
| **Portfolio Managers** | Faster convergence on high-conviction ideas; objective ranking of opportunities |
| **Family Offices** | Institutional-grade research capabilities without hiring a full analyst team |
| **Asset Managers** | Consistent, auditable investment decisions across the team |
| **Compliance/Chief Compliance Officers** | Every recommendation has an immutable, source-cited audit trail |
| **Wealth Advisors** | Evidence-backed recommendations to share with clients |

---

## The Investment Score

**Investment Score = Probability(Outperform S&P 500 over 252 trading days) × 100**

The Investment Score is a calibrated probability that a company will outperform the S&P 500 over the next 252 trading days (approximately one year). It is intended for **ranking opportunities** rather than estimating expected returns.

### What Drives the Score

For each company, the score is driven by **53 features** across 4 categories:

| Category | Features | Weight (Illustrative) |
|----------|----------|----------------------|
| **Fundamentals** | ROE, revenue growth, margins, leverage, liquidity | 40-50% |
| **Market** | Momentum, volatility, beta, drawdown | 25-35% |
| **Valuation** | P/E, P/B, P/S, EV/EBITDA | 15-20% |
| **Macro** | Interest rates, inflation, GDP | 5-10% |

**Note:** Feature weights vary by company, sector, and market regime. SHAP provides the exact contribution for each feature per prediction.

---

## How It Works: A Concrete Example

```
┌──────────────────────────────────────────────────────────────────────┐
│                    RECOMMENDATION EXAMPLE                           │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  COMPANY: Apple Inc. (AAPL)                                          │
│  INVESTMENT SCORE: 72/100                                            │
│                                                                       │
│  WHAT DROVE THE SCORE:                                               │
│  ────────────────────────────────────────────────────────────────── │
│  ✅ ROE: +15 points (147% — exceptional profitability)              │
│  ✅ Revenue Growth: +12 points (12.3% — strong top-line)           │
│  ❌ P/E Ratio: -8 points (34x — expensive valuation)               │
│  ❌ Volatility: -5 points (moderate risk)                          │
│                                                                       │
│  DOCUMENTARY EVIDENCE:                                               │
│  ────────────────────────────────────────────────────────────────── │
│  "Revenue grew 12.3% in fiscal 2023, driven by..."                 │
│  — Apple 10-K (2023), Section 1A, Page 12                          │
│                                                                       │
│  RISK METRICS:                                                       │
│  ────────────────────────────────────────────────────────────────── │
│  Beta: 1.2  │  Volatility: 22%  │  Max Drawdown: -18%              │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## The Scientific Foundation

CapitalQuant brings together three established ideas into a single investment decision framework:

| Idea | Source | Application |
|------|--------|-------------|
| **Gradient Boosting** | Chen & Guestrin (2016) — XGBoost | Handles non-linear relationships, missing values, and mixed data types in financial data without requiring scaling |
| **Game-Theoretic Explainability** | Lundberg & Lee (2017) — SHAP | Provides mathematically sound, additive feature attributions for every prediction |
| **Retrieval-Augmented Generation** | Lewis et al. (2020) | Enables verifiable, source-cited evidence extraction from SEC filings |

**"CapitalQuant does not invent anything new. It brings together three proven ideas into a single, transparent investment decision framework."**

---

## Walk-Forward Validation

The model is validated under realistic production conditions:

```
Window 1: Train [2010-2014] → Test [2015]
Window 2: Train [2011-2015] → Test [2016]
Window 3: Train [2012-2016] → Test [2017]
...
Window N: Train [2018-2022] → Test [2023]
```

### Realistic Performance Expectations

In quantitative finance, financial markets are highly stochastic. An AUC of **0.53 to 0.55** is often considered practically meaningful for directional stock prediction using public features.

| Metric | Target | Interpretation |
|--------|--------|----------------|
| **AUC-ROC** | > 0.55 | Better than random in noisy financial markets |
| **Precision@10** | > 0.55 | Top 10 recommendations outperform random |
| **Calibration** | Brier < 0.25 | Probabilities are well-calibrated |
| **Stability** | Std dev < 0.05 | Consistent performance across time |

**"The platform prioritizes robust MLOps, relative ranking, and explainability over absolute alpha generation. A slight edge, consistently applied, is the foundation of professional investing."**

---

## Competitive Positioning

| Traditional Screening | CapitalQuant |
|-----------------------|--------------|
| Static filters | Predictive ranking |
| Manual ratio analysis | ML + SHAP explanations |
| Read filings manually | RAG document search |
| Hard to audit | Fully reproducible, source-cited |
| Inconsistent across analysts | Consistent, objective scoring |
| Time-consuming | Automated, efficient |

---

## What CapitalQuant Is NOT

| ❌ NOT | ✅ IS |
|-------|-------|
| A magical prediction system | A systematic measure of current opportunity and risk |
| A buy/sell signal generator | An investment research assistant |
| A replacement for human analysts | A scalable second opinion that eliminates blind spots |
| Financial advice | A decision support tool |

**"CapitalQuant does not say 'buy this stock.' It says: 'Based on historical patterns, this company has a 72% probability of outperforming the market over the next year. Here's exactly what drove that score, and here's the documentary evidence to support it.'"**

---

## The Value Proposition

| Benefit | Description |
|---------|-------------|
| 🎯 **Objectivity** | Recommendations based on data and historical patterns, not intuition or bias |
| 🔍 **Transparency** | Each recommendation is explainable and broken down into individual factors |
| ⚡ **Efficiency** | Reduces time needed to filter and evaluate opportunities from hours to minutes |
| 📋 **Traceability** | Every decision can be backed by documentary evidence |
| 📊 **Auditability** | The system allows reconstruction of why each recommendation was made |
| 🔄 **Continuous Learning** | Automated retraining ensures the model adapts to changing market conditions |
| 🛡️ **Compliance-Ready** | Every recommendation has an immutable, source-cited audit trail |

---

## The Commercial Question

**"How do you know this isn't just overfitting?"**

> "Because I'm not predicting with a black box. I'm combining three proven ideas into a transparent framework. XGBoost provides state-of-the-art predictive performance on tabular data. SHAP ensures every prediction is explainable. And walk-forward validation prevents look-ahead bias — the model is always tested on data it has never seen before.
>
> When the model recommends a company, you don't just see the score. You see exactly what drove it. You can verify it against official documents. And you can reconstruct the decision at any time.
>
> That is not a black box. That is a transparent investment framework."

---

## Official Tagline

> **Predict. Explain. Verify.**

---

## Call to Action

**Let's run a blind test. Give us 5 tickers your team is currently debating, and we will generate a CapitalQuant report for you in 24 hours.**

---

**CapitalQuant combines prediction, explanation, and evidence — so every investment decision is transparent, auditable, and grounded in verifiable data.**

---

*CapitalQuant does not attempt to replace investment analysts. It augments their decision-making process by combining reliable financial data, explainable machine learning, and documentary evidence into a transparent investment research workflow.*