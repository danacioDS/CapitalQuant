# CapitalQuant

## AI-Powered Investment Decision Support Platform

---

## Product Concept Definition

---

**CapitalQuant** is an AI-powered investment decision support platform built to demonstrate **production-grade Machine Learning engineering**. Beyond predictive modeling, the platform showcases an **end-to-end MLOps architecture** including automated training pipelines, model versioning, deployment, monitoring, drift detection, explainability, and scalable cloud-native infrastructure.

The platform combines supervised Machine Learning models with SHAP explainability and augmented retrieval over official documents to prioritize investment opportunities in an objective, transparent, and evidence-backed manner — while operating as a **complete production system**.

---

## Context and Rationale

Investment professionals face three structural challenges that create demand for decision support tools:

| Challenge | Description |
|-----------|-------------|
| **Information overload** | Thousands of companies, hundreds of financial ratios, dozens of annual reports per company, plus news and macroeconomic data |
| **Cognitive biases** | Decisions influenced by market noise, emotions, or confirmation bias |
| **Lack of traceability** | Difficulty reconstructing and objectively explaining why a specific investment decision was made |

The platform addresses these problems by offering:

1. **Objective prioritization** — Based on historical patterns learned by the model
2. **Transparent explanations** — Each recommendation comes with a clear breakdown of contributing factors
3. **Documentary evidence** — Recommendations are backed by information extracted from official documents

---

## The Problem It Solves

Predictive models provide quantitative estimates, but rarely explain the reasoning behind a recommendation. The platform combines supervised learning, explainability, and documentary evidence retrieval to deliver transparent and auditable decisions, allowing the analyst to maintain final control.

For **ML Engineers and Data Scientists**, CapitalQuant serves as a reference implementation of:
- Production-grade ML system architecture
- MLOps best practices
- Model lifecycle management
- Continuous monitoring and drift detection

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRESENTATION LAYER                                 │
│                    Vercel (React Dashboard)                                │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           API GATEWAY                                      │
│                    Render (FastAPI + Rate Limiting + Auth)                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MODEL SERVING LAYER                                │
│                    ┌──────────┬──────────┬──────────┐                     │
│                    │Prediction│  SHAP   │   RAG    │                     │
│                    │ Service  │ Service │ Service  │                     │
│                    └──────────┴──────────┴──────────┘                     │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MLOPS INFRASTRUCTURE                               │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐             │
│  │  MLflow    │ │  Model     │ │ Prometheus │ │  Grafana   │             │
│  │  Tracking  │ │  Registry  │ │  Metrics   │ │  Dashboard │             │
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          FEATURE STORE                                     │
│               ┌────────────────────┬────────────────────┐                  │
│               │  Offline Store     │   Online Store     │                  │
│               │  (Neon PostgreSQL) │   (Redis Cloud)    │                  │
│               └────────────────────┴────────────────────┘                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          DATA INGESTION                                    │
│               ┌──────────┬──────────┬──────────┬──────────┐               │
│               │  Yahoo   │   SEC    │   FRED   │  S&P500  │               │
│               │ Finance  │  EDGAR   │          │  Const.  │               │
│               └──────────┴──────────┴──────────┴──────────┘               │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. Predictive Engine

The system uses **XGBoost** to predict which companies have the highest probability of outperforming the S&P500 over the next 252 trading days. The model is trained on fundamental and market data from the last 15 years using **walk-forward validation** to prevent data leakage and simulate real production conditions.

**Model Target:**
```
Target = 1 if Return_stock > Return_SP500
Target = 0 otherwise
```

**Realistic Performance Expectations:**
In quantitative finance, financial markets are highly stochastic and noisy. An AUC of **0.53 to 0.55** is often considered world-class for directional stock prediction. The system prioritizes **robust MLOps, relative ranking, and explainability** over absolute alpha generation. Minimum performance thresholds are dynamically set based on the baseline model's historical performance.

### 2. Explainability Engine

**SHAP** (SHapley Additive exPlanations) analyzes each prediction and decomposes the score into individual contributions from each financial variable, allowing the analyst to see exactly which factors (ROE, revenue growth, margin, etc.) positively or negatively influenced the recommendation.

**In production:** SHAP values are computed **asynchronously post-prediction** and cached in Redis to maintain API latency < 500ms. Value distributions are continuously monitored to detect feature drift and maintain explanation stability over time.

### 3. Document Engine

A **RAG** (Retrieval-Augmented Generation) system processes SEC 10-K and 10-Q reports, allowing analysts to ask natural language questions about the documents that support a recommendation. The system always cites exact sources (section and page of the original document).

**Financial Document Parsing:**
SEC filings contain complex multi-page tables and cross-referenced footnotes. Standard chunking destroys financial tables. The system uses specialized document parsers (e.g., LlamaParse or Unstructured.io) to retain table structure during chunking, ensuring accurate retrieval of numerical data from financial statements.

### 4. Presentation Layer

An interactive dashboard that enables:
- Viewing company rankings with their scores
- Exploring detailed explanations for each recommendation
- Querying official documents via natural language

---

## Production ML Architecture

The platform follows an end-to-end MLOps workflow designed for operational continuity:

### 4.1 Data Pipeline
- Automated ingestion from Yahoo Finance, SEC EDGAR, and FRED
- Data validation and quality checks
- Feature engineering pipeline with reproducibility
- Incremental updates for daily market data
- **Batch processing during off-peak hours** to minimize compute costs

### 4.2 Feature Store
- **Offline Store (Neon PostgreSQL):** Historical features for training
- **Online Store (Redis Cloud):** Low-latency features for inference
- Feature versioning for reproducibility
- Point-in-time correctness for time-series data
- **Custom lightweight implementation inspired by Feast**

### 4.3 Model Lifecycle Management
- **MLflow** for experiment tracking and model versioning
- **Model Registry** with staging → production promotion workflow
- Walk-forward validation for realistic performance assessment
- Automated testing before production deployment

### 4.4 Deployment Strategy
- **Docker** containerization for environment consistency
- **Render** for API deployment with auto-scaling
- **Vercel** for React dashboard deployment
- **FastAPI** for low-latency prediction serving
- Health checks and readiness probes
- **SHAP values computed asynchronously and cached in Redis** to ensure p95 latency < 500ms
- **RAG queries processed with timeout and fallback** to prevent API degradation

### 4.5 Monitoring & Observability
- **Prometheus** for metrics collection
- **Grafana** dashboards for real-time visualization
- Prediction latency tracking (p95, p99)
- API availability monitoring (uptime)
- Error rate tracking
- Resource utilization monitoring (CPU, Memory)
- **Structured Logging:** JSON logs with correlation IDs for request tracing

### 4.6 Drift Management
- **Data drift detection** — statistical distribution comparison (KS-test)
- **Concept drift monitoring** — rolling performance metrics (AUC)
- **Feature drift analysis** — SHAP value distribution shifts
- Automated alerting when drift exceeds thresholds
- Severity assessment (Low/Medium/High)

### 4.7 CI/CD Pipeline
- **GitHub Actions** for automated testing
- Container build and registry push
- Staging deployment validation
- Production deployment with approval gate
- Automated rollback on performance degradation

### 4.8 Security
- **Authentication:** JWT-based user authentication
- **Authorization:** Role-based access control (Analyst, Manager, Admin)
- **Secrets Management:** Environment variables / Doppler
- **API Security:** Rate limiting to prevent abuse
- **Data Encryption:** Encrypted at rest and in transit (TLS)

---

## Cloud-Native Infrastructure

```
                    ┌─────────────────┐
                    │   Internet      │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  Vercel         │
                    │  (React Dashboard)│
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  Render         │
                    │  (FastAPI API)  │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐
│  Neon         │   │  Redis Cloud  │   │  Docker Hub   │
│  PostgreSQL   │   │  (Cache)      │   │  (Registry)   │
└───────────────┘   └───────────────┘   └───────────────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  MLflow         │
                    │  (Tracking)     │
                    └─────────────────┘
```

---

## Infrastructure Components

| Component | Service | Purpose |
|-----------|---------|---------|
| **Frontend** | Vercel | React dashboard hosting with CDN |
| **Backend API** | Render | FastAPI service with auto-scaling |
| **Database** | Neon PostgreSQL | Serverless PostgreSQL for feature store |
| **Cache** | Redis Cloud | Low-latency online feature store + SHAP cache |
| **Container Registry** | Docker Hub | Model and service images |
| **ML Tracking** | MLflow | Experiment tracking and model registry |
| **Monitoring** | Prometheus + Grafana | Metrics collection and visualization |
| **CI/CD** | GitHub Actions | Automated testing and deployment |
| **Secrets** | Doppler / Env Vars | Secure credential management |

**Cost Management:**
- Batch processing scheduled during off-peak hours
- Open-source local embeddings for RAG to minimize API costs
- Free tiers utilized for Render, Vercel, Neon, and Redis Cloud during development
- Estimated monthly cost: **~$50-100** for production-like demo

---

## Monitoring Strategy

The system continuously monitors the following dimensions:

| Dimension | What is Measured | Alert Condition |
|-----------|------------------|-----------------|
| **Prediction Latency** | p95 response time (ms) | > 500ms |
| **API Availability** | Uptime percentage | < 99.9% |
| **Prediction Volume** | Requests per hour | Deviation > 3σ |
| **Error Rate** | Failed predictions % | > 1% |
| **Feature Distribution** | Statistical drift (KS-test) | p < 0.05 |
| **Model Performance** | Rolling AUC | Drop > 5% from baseline |
| **Data Quality** | Missing values, outliers | > 5% missing |
| **Resource Usage** | CPU, Memory, Network | > 80% utilization |

**Alerting & Response:**
- Email alerts for non-critical issues
- Slack/Discord webhooks for critical issues
- Automated rollback on performance degradation
- Incident playbooks documented and tested

---

## Drift Management Strategy

### Detection Methods

| Drift Type | Detection Method | Frequency |
|------------|------------------|-----------|
| **Data Drift** | Kolmogorov-Smirnov test | Daily |
| **Concept Drift** | Rolling performance metrics | Daily |
| **Feature Drift** | SHAP value distribution | Weekly |
| **Target Drift** | Target distribution change | Monthly |

### Response Protocol

```
1. Drift Detected → Alert Generated
2. Affected Features Identified
3. Severity Assessed (Low/Medium/High)
4. If High Severity:
   a. Model Retraining Triggered
   b. New Model Validated (walk-forward)
   c. Validated Model Promoted to Registry
   d. Canary Deployment to Production
   e. Performance Monitored (24h)
   f. Full Rollout or Rollback
```

### Automated Retraining

- Triggered by drift detection (Medium/High severity)
- Scheduled monthly as baseline
- Retraining window: last 3 years of data
- Validation: walk-forward on last year
- Minimum performance threshold: dynamically set based on baseline model performance
- Realistic threshold: AUC > 0.55 (financial markets are highly stochastic)

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
- **Realistic baseline:** AUC > 0.55 (financial markets are highly stochastic)

### RAG Evaluation
- Precision@5
- Recall@5
- Faithfulness
- Context Precision
- **Table extraction accuracy** for financial documents

### Production Evaluation
- API Latency (p95 < 500ms)
- System Uptime (> 99.9%)
- Drift Detection Rate
- Automated Retraining Success Rate

---

## Limitations

- Does not execute buy/sell orders
- Does not constitute financial advice
- Does not incorporate real-time news
- Relies on publicly available information
- Designed to support analysis, not replace the analyst
- **Does not account for transaction costs, slippage, or market impact** (critical in real portfolio construction)
- **Financial markets are highly stochastic** — the system prioritizes robust MLOps, relative ranking, and explainability over absolute alpha generation

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

As a **portfolio project**, CapitalQuant demonstrates the ability to design, build, deploy, and operate a complete ML system in production — from data ingestion to continuous monitoring and automated retraining.

---

## Differentiators

| Feature | Benefit |
|---------|---------|
| **Explainability as a core feature** | SHAP is not an add-on; it's fundamental to the product's value |
| **Integrated documentary evidence** | Recommendations backed by official documents, not just opinions |
| **Product architecture** | Designed as enterprise software, not a research notebook |
| **Walk-forward validation** | Model evaluated under realistic production-like conditions |
| **Production-ready infrastructure** | Containerization, monitoring, drift detection, CI/CD built-in |
| **Realistic ML expectations** | Appropriate thresholds for noisy financial data |

---

## Technical Roadmap

| Phase | Deliverable | Status |
|-------|-------------|--------|
| 1 | Data Ingestion Pipeline (Yahoo Finance, SEC, FRED) | Planned |
| 2 | Feature Engineering Pipeline | Planned |
| 3 | Model Training & Walk-forward Validation | Planned |
| 4 | SHAP Explainability Integration | Planned |
| 5 | MLflow Experiment Tracking & Model Registry | Planned |
| 6 | FastAPI Inference Service | Planned |
| 7 | Docker Containerization | Planned |
| 8 | Render Deployment Configuration | Planned |
| 9 | Vercel Frontend Deployment | Planned |
| 10 | Monitoring Dashboard (Prometheus + Grafana) | Planned |
| 11 | Drift Detection Implementation | Planned |
| 12 | Automated Retraining Pipeline | Planned |
| 13 | CI/CD Pipeline (GitHub Actions) | Planned |
| 14 | Production Demo & Documentation | Planned |

---

## Key Technologies

| Area | Technology |
|------|------------|
| **Machine Learning** | XGBoost, scikit-learn |
| **Explainability** | SHAP (asynchronous, cached) |
| **Document Intelligence** | RAG with LangChain, LlamaParse/Unstructured.io |
| **Experiment Tracking** | MLflow |
| **Model Registry** | MLflow Registry |
| **API Serving** | FastAPI, Python |
| **Containerization** | Docker |
| **Frontend Hosting** | Vercel |
| **Backend Hosting** | Render |
| **Database** | Neon PostgreSQL (Serverless) |
| **Cache / Online Store** | Redis Cloud |
| **Feature Store** | Custom lightweight implementation (inspired by Feast) |
| **Monitoring** | Prometheus |
| **Visualization** | Grafana |
| **Logging** | Structured JSON Logging with Correlation IDs |
| **CI/CD** | GitHub Actions |
| **Secrets Management** | Environment Variables / Doppler |
| **Workflow Orchestration** | Prefect / Airflow |
| **Security** | JWT, Role-based Access Control |
| **Frontend** | React, TypeScript |

---

## What CapitalQuant Demonstrates

| Skill | How It Is Demonstrated |
|-------|------------------------|
| **ML Engineering** | Complete training pipeline with walk-forward validation |
| **MLOps** | MLflow tracking, model registry, CI/CD |
| **Cloud-Native Deployment** | Vercel, Render, Neon PostgreSQL, Redis Cloud |
| **Monitoring** | Prometheus, Grafana, structured logging, alerting |
| **Drift Management** | Data drift, concept drift, automated retraining |
| **Scalability** | Auto-scaling on Render, Redis caching |
| **Production Autonomy** | Automated retraining decisions, rollback capabilities |
| **Security** | Authentication, authorization, rate limiting |
| **Feature Engineering** | Offline and online feature store |
| **Containerization** | Docker for consistency and portability |
| **Financial Document Parsing** | Specialized table extraction for 10-K/10-Q filings |
| **Latency Optimization** | Asynchronous SHAP computation with Redis caching |
| **Cost Awareness** | Batch processing, free tiers, open-source embeddings |

---

## Why Cloud-Native vs AWS

CapitalQuant uses a **cloud-native, provider-agnostic** infrastructure for three reasons:

| Reason | Benefit |
|--------|---------|
| **Portability** | The architecture can run on any cloud provider |
| **Focus on ML Engineering** | Effort goes into MLOps, not vendor-specific configurations |
| **Rapid Development** | Services like Render and Neon enable faster iteration |

The infrastructure principles demonstrated (containerization, CI/CD, monitoring, drift detection) are **transferable to any cloud environment**, including AWS, GCP, or Azure.

---

*CapitalQuant is a portfolio project designed to demonstrate production-grade ML engineering skills while solving a real-world investment decision support problem.*