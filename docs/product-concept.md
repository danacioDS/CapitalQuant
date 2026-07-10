# CapitalQuant

## AI-Powered Investment Decision Support

---

## Product Concept Definition

---

**CapitalQuant** is a decision support platform for financial analysts and portfolio managers that combines supervised Machine Learning models, SHAP explainability, and augmented retrieval over official documents to prioritize investment opportunities in an objective, transparent, and evidence-backed manner.

---

## Context and Rationale

Investment professionals face three structural challenges:

| Challenge | Description |
|-----------|-------------|
| **Information overload** | Thousands of companies, hundreds of financial ratios, dozens of annual reports per company, plus news and macroeconomic data |
| **Cognitive biases** | Decisions influenced by market noise, emotions, or confirmation bias |
| **Lack of traceability** | Difficulty reconstructing and objectively explaining why a specific investment decision was made |

The platform addresses these problems by offering:

1. **Objective prioritization** - Based on historical patterns learned by the model
2. **Transparent explanations** - Each recommendation comes with a clear breakdown of contributing factors
3. **Documentary evidence** - Recommendations are backed by information extracted from official documents

---

## The Problem It Solves

Predictive models provide quantitative estimates, but rarely explain the reasoning behind a recommendation. The platform combines supervised learning, explainability, and documentary evidence retrieval to deliver transparent and auditable decisions, allowing the analyst to maintain final control.

---

## Core Components

### 1. Predictive Engine

The system uses XGBoost to predict which companies have the highest probability of outperforming the S&P500 over the next 252 trading days. The model is trained on fundamental and market data from the last 15 years using walk-forward validation to prevent data leakage and simulate real production conditions.

**Model Target:**
```
Target = 1 if Return_stock > Return_SP500
Target = 0 otherwise
```

### 2. Explainability Engine

SHAP analyzes each prediction and decomposes the score into individual contributions from each financial variable, allowing the analyst to see exactly which factors (ROE, revenue growth, margin, etc.) positively or negatively influenced the recommendation.

### 3. Document Engine

A RAG system processes SEC 10-K and 10-Q reports, allowing analysts to ask natural language questions about the documents that support a recommendation. The system always cites exact sources (section and page of the original document).

### 4. Presentation Layer

An interactive dashboard that enables:
- Viewing company rankings with their scores
- Exploring detailed explanations for each recommendation
- Querying official documents via natural language

---

## Main Usage Flow

### Scenario: Investment Opportunity Review

A portfolio manager needs to identify new investment opportunities for the next quarter. They access the system and view a ranking of 500 companies ordered by Investment Score. To better understand the leaders, they select a company and the system displays:

1. The company's numerical score
2. A SHAP breakdown explaining what factors drove the score (e.g., "ROE of 32% contributed +25 points; P/E of 34x subtracted -8 points")
3. A document chat where they can ask: "What does the latest 10-K say about this company's growth?" and the system responds by extracting the relevant section from the report

The manager now has objective prioritization, a clear explanation of each opportunity, and direct access to documentary evidence, enabling more informed and defensible decisions.

---

## System Evaluation

### Model Evaluation
- Walk-forward validation (2010-2024)
- ROC AUC, Precision, Recall, Accuracy
- Calibration Curve
- Confusion Matrix
- Global and Local SHAP

### RAG Evaluation
- Precision@5
- Recall@5
- Faithfulness
- Context Precision

---

## Limitations

- Does not execute buy/sell orders
- Does not constitute financial advice
- Does not incorporate real-time news
- Relies on publicly available information
- Designed to support analysis, not replace the analyst

---

## User Value

| Benefit | Description |
|---------|-------------|
| **Objectivity** | Recommendations based on data and historical patterns, not intuition or bias |
| **Transparency** | Each recommendation is explainable and broken down into individual factors |
| **Efficiency** | Reduces time needed to filter and evaluate opportunities |
| **Traceability** | Every decision can be backed by documentary evidence |
| **Auditability** | The system allows reconstruction of why each recommendation was made |

---

## Positioning

The platform does not aim to replace the analyst's judgment, but rather to **amplify their analytical capability** by providing an objective and explainable intelligence layer upon which to make informed decisions.

---

## Differentiators

| Feature | Benefit |
|---------|---------|
| **Explainability as a core feature** | SHAP is not an add-on; it's fundamental to the product's value |
| **Integrated documentary evidence** | Recommendations backed by official documents, not just opinions |
| **Product architecture** | Designed as enterprise software, not a research notebook |
| **Walk-forward validation** | Model evaluated under realistic production-like conditions |

---

## Technical Roadmap

| Phase | Deliverable |
|-------|-------------|
| 1 | ETL from Yahoo Finance, FRED, and SEC EDGAR |
| 2 | Feature pipeline and reproducible dataset |
| 3 | XGBoost model with walk-forward validation |
| 4 | Global and local SHAP |
| 5 | FastAPI for predictions and explanations |
| 6 | 10-K ingestion, embeddings, and RAG |
| 7 | React Dashboard |
| 8 | Docker, README, and demo |

---

## Key Technologies

| Area | Technology |
|------|------------|
| Machine Learning | XGBoost, scikit-learn |
| Explainability | SHAP |
| Document Intelligence | RAG with LangChain, ChromaDB |
| Backend | FastAPI, Python |
| Frontend | React, TypeScript |
| Containerization | Docker |
| MLOps | MLflow |
