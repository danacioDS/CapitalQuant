# Data Acquisition Strategy: CapitalQuant

**How do we source, validate, and maintain data for explainable AI investment research?**

---

## Executive Summary

CapitalQuant transforms heterogeneous financial information into **reproducible, explainable, and point-in-time correct investment intelligence**. Rather than competing with traditional financial data vendors, the platform augments them through an end-to-end AI pipeline that combines validated market data, corporate fundamentals, macroeconomic indicators, and documentary evidence into transparent investment recommendations.

Every data source must contribute either **predictive value**, **explanatory evidence**, or **model validation**. The platform is designed so that every recommendation can be traced back to its underlying evidence, allowing analysts to understand not only what the model predicts, but **why** it predicts it.

**The entire platform is built using free, open-source technologies and freely available data sources**—demonstrating that sophisticated investment intelligence can be constructed without expensive enterprise contracts or proprietary software.

---

## Technology Stack

| Layer | Technology | License | Rationale |
|-------|------------|---------|-----------|
| **API Serving** | FastAPI | MIT | High-performance, async, auto-documentation |
| **Data Validation** | Great Expectations | Apache 2.0 | Industry standard for data quality |
| **Workflow Orchestration** | Prefect | Apache 2.0 | Modern, Python-native, simpler than Airflow |
| **Storage (OLTP)** | PostgreSQL | PostgreSQL License | Reliable, ACID-compliant, free |
| **Cache / Online Store** | Redis | BSD | Low-latency, battle-tested |
| **Vector Database** | pgvector | PostgreSQL Extension | Native PostgreSQL, no new infrastructure |
| **ML Framework** | XGBoost | Apache 2.0 | Production-grade, interpretable |
| **Explainability** | SHAP | MIT | Industry standard for model explanations |
| **Document Parsing** | Unstructured.io | Apache 2.0 | Open-source, financial document support |
| **Embeddings** | all-MiniLM-L6-v2 | MIT | Lightweight, open-source, runs locally |
| **Frontend** | React + TypeScript | MIT | Modern, widely adopted |
| **Visualization** | Plotly + D3 | MIT | Interactive charts for SHAP and returns |
| **Deployment** | Docker | Apache 2.0 | Containerization, portability |
| **CI/CD** | GitHub Actions | Proprietary (free) | Free for open-source |
| **Infrastructure** | Self-hosted / Fly.io / Railway | Various | Low-cost, open-source friendly |
| **Monitoring** | Prometheus + Grafana | Apache 2.0 | Industry standard observability |
| **Drift Detection** | Evidently AI | Apache 2.0 | Open-source ML monitoring |
| **Feature Store** | Feast (optional) | Apache 2.0 | Open-source feature store |
| **Secrets Management** | Vault / Doppler (free tier) | Business Source | Secure credential management |
| **Logging** | ELK Stack (optional) | Elastic License | Open-source logging |

### Cost Estimation

| Resource | Service | Monthly Cost |
|----------|---------|--------------|
| **Database** | PostgreSQL (self-hosted or free tier) | $0 - $15 |
| **Cache** | Redis (free tier or self-hosted) | $0 - $10 |
| **Compute** | Fly.io / Railway (free tier) | $0 - $20 |
| **Storage** | S3-compatible (free tier) | $0 - $10 |
| **CI/CD** | GitHub Actions | $0 |
| **Monitoring** | Prometheus + Grafana (self-hosted) | $0 |
| **Total** | | **$0 - $55/month** |

---

## Conceptual Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    CONCEPTUAL DATA FLOW                             │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │                    DATA SOURCES                                 │ │
│  │                    (All Free)                                   │ │
│  │                                                                  │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │ │
│  │  │   Stooq     │  │  SEC XBRL   │  │  FRED API   │           │ │
│  │  │ AlphaVantage│  │  SEC EDGAR  │  │             │           │ │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘           │ │
│  │         │                │                │                   │ │
│  │  ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐           │ │
│  │  │   Market    │  │  Corporate  │  │Macroeconomic│           │ │
│  │  │  Behavior   │  │Fundamentals │  │  Context    │           │ │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘           │ │
│  │         │                │                │                   │ │
│  │         └────────────────▼────────────────┘                   │ │
│  │                         │                                      │ │
│  │  ┌──────────────────────▼─────────────────────────┐           │ │
│  │  │      DATA CONTRACTS + VALIDATION               │           │ │
│  │  │      (Great Expectations)                      │           │ │
│  │  └──────────────────────┬─────────────────────────┘           │ │
│  │                         │                                      │ │
│  │  ┌──────────────────────▼─────────────────────────┐           │ │
│  │  │         FEATURE STORE (PostgreSQL + Redis)      │           │ │
│  │  │   (Offline: Training │ Online: Inference)      │           │ │
│  │  └──────────────────────┬─────────────────────────┘           │ │
│  │                         │                                      │ │
│  │  ┌──────────────────────▼─────────────────────────┐           │ │
│  │  │      MACHINE LEARNING + SHAP (XGBoost)         │           │ │
│  │  │   (Predictions with SHAP explanations)         │           │ │
│  │  └──────────────────────┬─────────────────────────┘           │ │
│  │                         │                                      │ │
│  │  ┌──────────────────────▼─────────────────────────┐           │ │
│  │  │   DOCUMENT INTELLIGENCE (RAG) - pgvector       │           │ │
│  │  │     (SEC filings → Evidence → Citations)       │           │ │
│  │  └──────────────────────┬─────────────────────────┘           │ │
│  │                         │                                      │ │
│  │  ┌──────────────────────▼─────────────────────────┐           │ │
│  │  │       INVESTMENT DECISION + MONITORING         │           │ │
│  │  │   (Prometheus + Grafana + Evidently AI)        │           │ │
│  │  └─────────────────────────────────────────────────┘           │ │
│  │                                                                  │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Design Principles

| Principle | What It Means | Why It Matters |
|-----------|---------------|----------------|
| **Provider Agnostic** | No vendor lock-in; data sources can be swapped | Allows migration without redesign |
| **Open Source First** | All tools must be open-source and freely available | Zero licensing costs, community support |
| **Explainability First** | Every dataset must enable SHAP-based explanations | Analysts need to understand **why** |
| **Point-in-Time Correctness** | Data is always as-of the report date | Prevents look-ahead bias |
| **Bitemporal Data Model** | Track both when data is valid and when it's known | Handles restatements correctly |
| **Reproducibility** | Every model can be retrained with identical data | Enables audits and compliance |
| **Idempotent Pipelines** | Repeated executions produce identical outputs | Prevents data corruption |
| **Modular Pipelines** | Each source is independent and self-contained | Allows independent upgrades |
| **Cost-Aware MVP** | Free sources first, enterprise later | Maximizes speed while controlling costs |
| **Self-Hosted First** | Prefer self-hosted over SaaS where possible | Maximizes control and minimizes cost |

---

## Data Sources

### All Free and Legally Compliant

| Source | Data Provided | Cost | License | Production Ready |
|--------|---------------|------|---------|------------------|
| **SEC EDGAR** | 10-K, 10-Q, 8-K filings, proxy statements | Free | Public Domain | ✅ |
| **SEC XBRL API** | Standardized financial statements | Free | Public Domain | ✅ |
| **FRED API** | Interest rates, CPI, unemployment, GDP | Free | Public Domain | ✅ |
| **Alpha Vantage** | Daily OHLCV prices, technical indicators | Free tier | Proprietary API | ✅ |
| **Stooq** | Historical OHLCV prices (global coverage) | Free | Free for use | ✅ |
| **Financial Modeling Prep** | Company fundamentals, ratios | Free tier | Proprietary API | ⚠️ |
| **S&P Constituents** | S&P 500 membership (Wikipedia) | Free | CC BY-SA | ✅ |
| **OpenFIGI API** | Security identifiers | Free | Open | ✅ |
| **SEC EDGAR (RAG)** | Full 10-K/10-Q text | Free | Public Domain | ✅ |

### Why Each Source Exists

| Source | Primary Purpose | Contributes To |
|--------|-----------------|----------------|
| **SEC EDGAR** | Corporate fundamentals and documentary evidence | Prediction + Evidence |
| **SEC XBRL** | Standardized, structured financials | Prediction + Explanation |
| **Alpha Vantage / Stooq** | Market behavior (prices, volumes, returns) | Prediction + Explanation |
| **FRED API** | Macroeconomic context | Prediction + Explanation |
| **Financial Modeling Prep** | Supplementary ratios for validation | Validation |
| **OpenFIGI** | Identifier mapping across sources | Reliability |
| **S&P Constituents** | Survivorship correction for validation | Validation |

---

## Medallion Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    MEDALLION ARCHITECTURE                           │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │                    BRONZE (Raw Zone)                            │ │
│  │                                                                  │ │
│  │  Purpose: Immutable audit trail and reprocessing               │ │
│  │  Format: Original source format (JSON, CSV, HTML, XBRL)        │ │
│  │  Technology: Object Storage (S3-compatible / MinIO)            │ │
│  │  Retention: 15 years                                           │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    SILVER (Validated Zone)                     │ │
│  │                                                                  │ │
│  │  Purpose: Clean, validated data ready for analysis             │ │
│  │  Technology: PostgreSQL                                        │ │
│  │  Retention: 15 years                                           │ │
│  │  Validation: Great Expectations                                │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    GOLD (Curated Zone)                        │ │
│  │                                                                  │ │
│  │  Purpose: Feature-engineered data for model consumption        │ │
│  │  Technology: PostgreSQL (Offline) + Redis (Online)            │ │
│  │  Retention: 15 years                                           │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Data Contracts

### What Is a Data Contract?

A data contract is a formal agreement between data producers and consumers that defines:

- **Schema**: Expected fields, types, and formats
- **Quality**: Acceptable null rates, value ranges, and business rules
- **SLA**: Freshness, availability, and latency guarantees
- **Versioning**: How changes are communicated and managed

### Data Contract Example

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DATA CONTRACT EXAMPLE                             │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  Contract: alpha_vantage_prices_v1                              │ │
│  │  ────────────────────────────────────────────────────────────── │ │
│  │  Producer: Alpha Vantage ETL Pipeline                         │ │
│  │  Consumer: Feature Engineering Pipeline                        │ │
│  │                                                                  │ │
│  │  SCHEMA                                                          │ │
│  │  ────────────────────────────────────────────────────────────── │ │
│  │  ticker:    string, required, valid S&P 500 symbol             │ │
│  │  date:      date, required, format YYYY-MM-DD                  │ │
│  │  open:      float, required, range (0, 10000)                  │ │
│  │  high:      float, required, range (0, 10000), >= open         │ │
│  │  low:       float, required, range (0, 10000), <= open         │ │
│  │  close:     float, required, range (0, 10000)                  │ │
│  │  volume:    integer, required, >= 0                            │ │
│  │                                                                  │ │
│  │  QUALITY SLA                                                    │ │
│  │  ────────────────────────────────────────────────────────────── │ │
│  │  Null rate: < 1% per column                                    │ │
│  │  Completeness: > 99% of expected records                      │ │
│  │  Freshness: < 24 hours since last update                      │ │
│  │                                                                  │ │
│  │  VERSIONING                                                     │ │
│  │  ────────────────────────────────────────────────────────────── │ │
│  │  Current: v1.2                                                  │ │
│  │  Deprecation notice: 30 days for breaking changes             │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Idempotent Pipelines

### Principle

> Pipeline runs are idempotent. Repeated executions produce identical outputs for the same source version.

### Implementation

| Property | Implementation |
|----------|----------------|
| **Deterministic Execution** | Same inputs produce same outputs |
| **Source Versioning** | Each source data version is immutable |
| **Output Versioning** | Each pipeline run produces a versioned output |
| **Exactly-Once Semantics** | Duplicate runs don't create duplicate outputs |
| **Checkpointing** | Pipeline can resume from last successful checkpoint |

---

## SEC XBRL Advantage

### Why SEC XBRL Changes Everything

```
┌──────────────────────────────────────────────────────────────────────┐
│                    SEC XBRL ADVANTAGE                               │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  TRADITIONAL APPROACH (PDF Parsing)                                 │
│  ────────────────────────────────────────────────────────────────── │
│  SEC Filing (PDF/HTML)                                              │
│        ↓                                                            │
│  Table Extraction (Error-prone)                                    │
│        ↓                                                            │
│  Manual Validation                                                 │
│        ↓                                                            │
│  Financial Ratios                                                  │
│                                                                       │
│  ⚠️ Problems:                                                       │
│  • Parsing errors                                                   │
│  • Formatting inconsistencies                                       │
│  • Labor-intensive validation                                      │
│                                                                       │
│  MODERN APPROACH (XBRL)                                             │
│  ────────────────────────────────────────────────────────────────── │
│  SEC Filing (XBRL)                                                  │
│        ↓                                                            │
│  SEC XBRL API (Free)                                                │
│        ↓                                                            │
│  Structured Data:                                                   │
│  • Revenue, Net Income, Assets, Liabilities                        │
│  • Cash Flow, Equity, EPS, etc.                                   │
│                                                                       │
│  ✅ Advantages:                                                      │
│  • Machine-readable, official SEC data                              │
│  • Standardized taxonomy, free                                     │
│  • Point-in-time correct                                           │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Point-in-Time & Bitemporal Data Model

### Why It Matters

```
┌──────────────────────────────────────────────────────────────────────┐
│                    POINT-IN-TIME CORRECTNESS                        │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  PROBLEM: Look-Ahead Bias                                            │
│  Using information not yet available at prediction time             │
│  Overstates model performance                                       │
│                                                                       │
│  SOLUTION: Bitemporal Data Model                                    │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  Financial Statement Released (Q4 2023)                         │ │
│  │  valid_at: 2023-10-01 to 2023-12-31                           │ │
│  │  known_at: 2024-02-01 (filing date)                           │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  Restatement Filed (10-K/A, Q4 2023 Correction)                │ │
│  │  valid_at: 2023-10-01 to 2023-12-31 (same period)             │ │
│  │  known_at: 2024-03-15 (amended filing date)                   │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  Model Training (Walk-Forward)                                  │ │
│  │  Prediction Date: 2024-02-15                                   │ │
│  │  Data Available: All reports where known_at <= 2024-02-15     │ │
│  │  Data NOT Available: Restatement (known_at: 2024-03-15)       │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

### Bitemporal Schema

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `ticker` | string | Company identifier | AAPL |
| `valid_at_start` | date | Start of reporting period | 2023-10-01 |
| `valid_at_end` | date | End of reporting period | 2023-12-31 |
| `known_at` | datetime | When data became public | 2024-02-01 16:30:00 |
| `feature_name` | string | Financial metric | roe |
| `feature_value` | float | Value as-reported | 0.31 |
| `restated` | boolean | Whether data was amended | false |

---

## Data Governance

### Governance Framework

| Aspect | Implementation | Why It Matters |
|--------|----------------|----------------|
| **Data Ownership** | Each source owned by a data engineer | Clear accountability |
| **Data Stewardship** | Quality monitoring assigned to stewards | Continuous quality oversight |
| **Metadata Management** | Complete schema documentation | Discoverability |
| **Data Lineage** | Every data point traceable to source | Audit and compliance |
| **Retention Policy** | 15 years for regulatory compliance | Reproducibility |
| **Access Policies** | Role-based access controls | Security |
| **Versioning** | Each dataset versioned and immutable | Rollback capability |

### Data Lineage Example

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DATA LINEAGE EXAMPLE                             │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  Source: SEC EDGAR (XBRL)                                            │
│  ────────────────────────────────────────────────────────────────── │
│  Filing: Apple 10-K (2023)                                          │
│  Accession: 0000320193-23-000106                                    │
│  XBRL Extraction: 2024-01-15 02:30:00 UTC                         │
│  Version: v1.2.3                                                    │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  Processing: XBRL Parsing (Great Expectations Validation)           │
│  ────────────────────────────────────────────────────────────────── │
│  Concept: Revenue                                                   │
│  Value: $383,285,000                                               │
│  Period: 2023-10-01 to 2023-12-31                                 │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  Storage: Silver Zone (PostgreSQL)                                  │
│  ────────────────────────────────────────────────────────────────── │
│  Table: xbrl_facts.apple_2023_q4                                   │
│  Checksum: 8a7f3b2c...                                             │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  Model Input: Gold Zone (Feature Store)                            │
│  ────────────────────────────────────────────────────────────────── │
│  Feature: Revenue_Growth = 12.3%                                   │
│  valid_at: 2023-10-01 to 2023-12-31                               │
│  known_at: 2024-02-01 16:30:00                                    │
│  Used in: Model v2.3.1 (XGBoost)                                   │
│  Prediction: 2024-06-01                                            │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Document Intelligence (RAG)

### RAG Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    RAG ARCHITECTURE                                 │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │                    DOCUMENT INGESTION                            │ │
│  │  SEC EDGAR 10-K/10-Q → Unstructured.io → Text + Tables        │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    CHUNKING & EMBEDDING                        │ │
│  │  • Chunks: 1,024 tokens, 128 overlap                          │ │
│  │  • Table chunks: Preserved structure                          │ │
│  │  • Embedding: all-MiniLM-L6-v2 (open-source, local)          │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    VECTOR STORE (pgvector)                     │ │
│  │  • Hybrid search (semantic + keyword)                         │ │
│  │  • Metadata filters (year, filing type, section)             │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    RETRIEVAL & RERANKING                       │ │
│  │  1. Semantic search + keyword boost                           │ │
│  │  2. Cross-encoder reranking                                   │ │
│  │  3. Return top-5 with source citations                        │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

### Table Extraction

| Approach | Problem | Solution |
|----------|---------|----------|
| **Standard chunking** | Tables flattened, structure lost | Unstructured.io preserves structure |
| **HTML parsing** | Missing context, footnotes lost | Parse with row/column headers |

---

## Feature Store Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    FEATURE STORE ARCHITECTURE                       │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │                    OFFLINE STORE (PostgreSQL)                   │ │
│  │  Purpose: Training, validation, and backtesting                │ │
│  │  Schema: ticker, valid_at, known_at, feature_1...feature_N   │ │
│  │  Retention: 15 years                                           │ │
│  └────────────────────────┬───────────────────────────────────────┘ │
│                           │                                          │
│  ┌────────────────────────▼──────────────────────────────────────┐ │
│  │                    ONLINE STORE (Redis)                        │ │
│  │  Purpose: Low-latency inference                                │ │
│  │  Schema: Key: ticker, Value: feature vector                   │ │
│  │  TTL: 24 hours                                                 │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Data Observability

| Metric | What It Measures | Alert Threshold |
|--------|------------------|-----------------|
| **Freshness** | Time since last successful update | > 24 hours |
| **Completeness** | Percentage of expected records | < 98% |
| **Error Rate** | Percentage of failed extraction jobs | > 1% |
| **Null Rate** | Percentage of null values per feature | > 5% |
| **Distribution Drift** | Change in feature distributions | KS p < 0.05 |
| **Schema Changes** | Column additions/deletions | Any change |
| **Latency** | Time to complete pipeline | > 2 hours |

---

## MLOps Lifecycle

```
┌──────────────────────────────────────────────────────────────────────┐
│                    MLOPS LIFECYCLE                                   │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  DATA INGESTION                                                      │
│  ────────────────────────────────────────────────────────────────── │
│  • Alpha Vantage, SEC XBRL, FRED, SEC EDGAR                       │
│  • Prefect orchestration                                            │
│  • Great Expectations validation                                   │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  FEATURE STORE                                                       │
│  ────────────────────────────────────────────────────────────────── │
│  • Offline: PostgreSQL for training                                │
│  • Online: Redis for inference                                     │
│  • Feature versioning                                              │
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
│  MODEL REGISTRY                                                      │
│  ────────────────────────────────────────────────────────────────── │
│  • Versioned models stored                                          │
│  • Staging → Production promotion                                  │
│  • Rollback capability                                              │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  DEPLOYMENT                                                          │
│  ────────────────────────────────────────────────────────────────── │
│  • Docker containers                                               │
│  • FastAPI serving                                                 │
│  • Health checks                                                   │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  MONITORING                                                          │
│  ────────────────────────────────────────────────────────────────── │
│  • Prometheus + Grafana                                            │
│  • Evidently AI for drift                                          │
│  • Latency, error rate, throughput                                │
│                                                                       │
│  ↓                                                                   │
│                                                                       │
│  RETRAINING                                                          │
│  ────────────────────────────────────────────────────────────────── │
│  • Triggered by drift (KS p < 0.05) or monthly                    │
│  • Walk-forward validation                                         │
│  • Canary deployment                                               │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Failure Scenarios

### Scenario 1: Alpha Vantage Unavailable

```
Alpha Vantage API Times Out
        │
        ▼
Retry (exponential backoff)
        │
        ▼
Fallback to Stooq
        │
        ▼
Continue Pipeline
        │
        ▼
Alert Sent: "Data source degraded"
        │
        ▼
Mark Data as "Degraded" in Silver Zone
        │
        ▼
Analyst Notified (optional)
```

### Scenario 2: SEC API Timeout

```
SEC API Timeout
        │
        ▼
Retry x3 (exponential backoff)
        │
        ▼
Partial Ingestion (successful tickers only)
        │
        ▼
Mark as "Partial" with missing tickers
        │
        ▼
Alert Sent: "SEC ingestion partial"
        │
        ▼
Manual Intervention if > 5% missing
```

### Scenario 3: Data Quality Violation

```
Null Rate > 5% in Feature
        │
        ▼
Pipeline Continues (soft failure)
        │
        ▼
Alert Sent: "Data Quality Warning"
        │
        ▼
Feature Marked as "Degraded" in Catalog
        │
        ▼
Investigation Triggered
```

---

## Security

| Aspect | Implementation |
|--------|----------------|
| **Secrets Management** | Vault / Doppler (free tier) |
| **API Keys** | Rotated quarterly, stored in secrets manager |
| **Data Encryption** | TLS in transit, encryption at rest |
| **Access Control** | Role-based access (Analyst, Engineer, Admin) |
| **Audit Logs** | All data access logged |
| **IAM** | Service accounts with least privilege |

---

## MVP Scope

```
┌──────────────────────────────────────────────────────────────────────┐
│                        MVP SCOPE                                     │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  SOURCES (All Free)                                                  │
│  ────────────────────────────────────────────────────────────────── │
│  • Alpha Vantage / Stooq (market data)                             │
│  • SEC EDGAR (corporate filings for RAG)                          │
│  • SEC XBRL API (structured fundamentals)                         │
│  • FRED API (macroeconomic data)                                  │
│  • Financial Modeling Prep (supplementary ratios)                 │
│  • S&P 500 constituents (Wikipedia)                               │
│                                                                       │
│  TECHNOLOGY (All Open Source)                                       │
│  ────────────────────────────────────────────────────────────────── │
│  • FastAPI (API serving)                                           │
│  • PostgreSQL + Redis (Feature Store)                              │
│  • pgvector (Vector database)                                      │
│  • Great Expectations (Data validation)                           │
│  • Prefect (Orchestration)                                         │
│  • XGBoost + SHAP (ML + Explainability)                           │
│  • Unstructured.io (Document parsing)                             │
│  • all-MiniLM-L6-v2 (Embeddings)                                  │
│  • Prometheus + Grafana (Monitoring)                              │
│  • Evidently AI (Drift detection)                                 │
│  • Docker (Deployment)                                             │
│  • GitHub Actions (CI/CD)                                          │
│                                                                       │
│  COVERAGE                                                             │
│  • S&P 500 companies (500)                                          │
│  • 15 years of historical data                                      │
│  • 50+ financial ratios                                             │
│  • 10-K and 10-Q filings for RAG                                  │
│                                                                       │
│  COST                                                                 │
│  • Estimated monthly: $0 - $55 (free tiers + minimal compute)       │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Summary

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DATA ACQUISITION STRATEGY SUMMARY                │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  PHILOSOPHY: Quality data + rigorous validation + transparent       │
│              explanations = investment intelligence                 │
│                                                                       │
│  SOURCES: All free (SEC XBRL, FRED, Alpha Vantage, Stooq, etc.)    │
│                                                                       │
│  TECH STACK: 100% Open Source                                       │
│  │  FastAPI, PostgreSQL, Redis, pgvector, Great Expectations,      │ │
│  │  Prefect, XGBoost, SHAP, Unstructured.io, all-MiniLM,          │ │
│  │  Prometheus, Grafana, Evidently AI, Docker, GitHub Actions     │ │
│                                                                       │
│  COST: $0 - $55/month                                                │
│                                                                       │
│  ARCHITECTURE: Medallion (Bronze → Silver → Gold)                 │
│                                                                       │
│  PRINCIPLES: Provider Agnostic, Explainability First,              │
│             Point-in-Time, Bitemporal, Reproducible, Idempotent   │
│                                                                       │
│  MVP: S&P 500, 50+ features, XBRL fundamentals, RAG,              │
│       15 years data, monitoring, drift detection, CI/CD           │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

*CapitalQuant does not attempt to replace investment analysts. It augments their decision-making process by combining reliable financial data, explainable machine learning, and documentary evidence into a transparent investment research workflow. Every recommendation remains reproducible, auditable, and grounded in verifiable data.*