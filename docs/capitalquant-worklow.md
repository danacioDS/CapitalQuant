# CapitalQuant Development Methodology — Complete Workflow

Your methodology flows from **why** (conceptual) → **what** (design) → **how** (implementation) in six distinct phases.

---

## Phase 0: Conceptual Foundation

**Purpose:** Establish the investment thesis, economic rationale, and strategic positioning before any design or code.

**Documents (already complete):**

| # | Document | Question It Answers |
|---|----------|---------------------|
| 01 | Commercial Pitch | Why does this product need to exist? Who pays for it? |
| 02 | Investment Theory | What behavioral finance principles justify the approach? |
| 03 | Statistics Theory | How do we measure outperformance? Why XGBoost? |
| 04 | Commercial Strategy | How does this compete? What's the moat? |
| 05 | Data Acquisition | What data sources? Why these sources? |
| 06 | Operational Strategy | How does this scale to multiple sectors? |

**Gate condition:** Conceptual docs reviewed and approved by stakeholders.

**Output:** Confidence that the product idea is sound before spending engineering time.

---

## Phase 1: High-Level Design (HLD)

**Purpose:** Translate conceptual foundation into system architecture. Define layers, data flows, and component boundaries without implementation details.

**Documents:**
- `docs/hld/CapitalQuant_mvp_plan.md` — Architecture, data flow diagrams, technology choices
- `docs/hld/product_engineering.md` — Feature scope, MVP boundaries, post-MVP roadmap
- `docs/hld/product_expectations.md` — Output specifications, validation criteria, success metrics

**Key decisions made here:**
- Production-grade ML architecture (Ingestion → Feature Store → ML → Serving → Monitoring → UI)
- XGBoost as core model with walk-forward validation
- SHAP as explainability engine (core feature, not optional)
- Feature Store: Neon PostgreSQL (Offline) + Redis Cloud (Online)
- Model Registry: MLflow with staging → production promotion
- Deployment: Docker + Render (API) + Vercel (Frontend)
- Monitoring: Prometheus + Grafana
- Drift Management: Data drift + Concept drift + Feature drift
- RAG: LangChain with specialized financial document parsing (LlamaParse/Unstructured.io)
- CI/CD: GitHub Actions with automated testing and deployment

**Gate condition:** HLD reviewed, dependencies identified, resource estimates approved.

**Output:** System block diagram with clear layer boundaries and MLOps workflow.

---

## Phase 2: Low-Level Design (LLD)

**Purpose:** Detailed technical specifications for each layer. Function signatures, API contracts, database schemas, error handling rules.

**Documents (one per layer):**

| Layer | Document | Focus |
|-------|----------|-------|
| **L1** | `lld/layer_01_data_ingestion.md` | Yahoo Finance, SEC EDGAR, FRED — ETL with validation |
| **L2** | `lld/layer_02_feature_store.md` | Neon PostgreSQL schema, Redis schema, point-in-time correctness |
| **L3** | `lld/layer_03_feature_engineering.md` | 50+ features, transformations, versioning |
| **L4** | `lld/layer_04_ml_engine.md` | XGBoost training, walk-forward validation, SHAP integration |
| **L5** | `lld/layer_05_document_intelligence.md` | RAG pipeline, chunking strategy, table extraction |
| **L6** | `lld/layer_06_serving_api.md` | FastAPI endpoints, async SHAP, Redis caching |
| **L7** | `lld/layer_07_monitoring_drift.md` | Prometheus metrics, drift detection, alerting rules |
| **L8** | `lld/layer_08_ci_cd.md` | GitHub Actions workflows, staging/production gates |
| **L9** | `lld/layer_09_ui_dashboard.md` | React components, SHAP visualizations, RAG chat |

**Each LLD includes:**
- Function signatures with parameter types and return types
- Expected behavior for happy path and all error conditions
- Data schemas (JSON, SQL DDL, or both)
- Test criteria (what must be tested)
- Dependencies on other layers
- Performance targets (latency, throughput)

**Gate condition:** LLD reviewed for completeness, no ambiguities, all edge cases covered.

**Output:** A document that an independent engineer could implement from without asking questions.

---

## Phase 3: Production Specification

**Purpose:** A single, frozen master document that unifies HLD and LLD into an executable specification. This is the "contract" between design and implementation.

**Document:** `docs/spec/CapitalQuant_production_spec.md`

**What it contains:**
- Executive summary (one page)
- Complete architecture diagram with all layers
- Layer-by-layer specifications (synthesized from HLD + LLD)
- Data flow descriptions (training and inference pipelines)
- Validation criteria (statistical tests, acceptance criteria)
- Success metrics:
  - Model: AUC > 0.55 (realistic for financial data)
  - API: p95 latency < 500ms
  - Uptime: > 99.9%
  - RAG: Faithfulness > 0.8
  - Drift Detection: Alert rate < 5% weekly
- Deployment architecture (Render, Vercel, Neon, Redis Cloud)
- Monitoring and alerting configuration
- Disaster recovery and rollback procedures

**Why this exists:** The unified spec prevents HLD/LLD drift. When HLD and LLD conflict, the unified spec is truth.

**Gate condition:** Unified spec frozen. No changes without formal change request.

**Output:** A single document that drives all implementation prompts.

---

## Phase 4: Prompt Generation

**Purpose:** Convert the unified spec into executable prompts for each module. Each prompt is self-contained and testable.

**Documents:** `docs/prompts/prompts_layer_XX.md`

**Prompt structure:**

| Section | Content |
|---------|---------|
| Overview | What this module does, why it exists |
| Input specification | Function signatures, expected data shapes |
| Output specification | Return values, side effects |
| Error handling | Table of error types → actions |
| Retry logic | If applicable |
| CLI | Command-line interface if applicable |
| Test criteria | What must be tested |
| Example | Input/output example |

**Gate condition:** Prompts reviewed for clarity, no ambiguities, testable.

**Output:** A set of prompts that a coding agent (human or LLM) can execute in order.

---

## Phase 5: Implementation

**Purpose:** Execute prompts to produce code, tests, and deployment artifacts.

**Process for each layer:**
1. Execute Prompt 1 → module 1 → test → commit
2. Execute Prompt 2 → module 2 → test → commit
3. ...continue through all prompts for the layer
4. Run full layer test suite
5. Verify no regressions in existing layers
6. Tag release (e.g., `v0.2-with-layer1`)

**Order of implementation (recommended for CapitalQuant):**

| Priority | Layer | Why This Order |
|----------|-------|----------------|
| **1** | L4: ML Engine | Proves the investment thesis works with synthetic data |
| **2** | L3: Feature Engineering | Provides real features to the ML Engine |
| **3** | L2: Feature Store | Stores features for reproducibility |
| **4** | L1: Data Ingestion | Feeds real data into the pipeline |
| **5** | L6: Serving API | Exposes predictions via FastAPI |
| **6** | L5: Document Intelligence | Adds RAG capabilities |
| **7** | L7: Monitoring & Drift | Adds production observability |
| **8** | L8: CI/CD | Automates deployment |
| **9** | L9: UI Dashboard | Presents everything to users |

**Gate condition:** All tests pass, no regressions, synthetic demo works.

**Output:** Working code, passing tests, deployment artifacts.

---

## Phase 6: Validation

**Purpose:** Verify that the implemented system meets the success criteria defined in conceptual and HLD phases.

**Validation activities:**

| Component | Validation Metrics | Target |
|-----------|-------------------|--------|
| **ML Model** | Walk-forward validation, ROC-AUC | AUC > 0.55 (realistic for finance) |
| **ML Model** | Precision, Recall, Accuracy | Baseline + 5% |
| **SHAP Explanations** | Global consistency, Local stability | Stable across windows |
| **RAG System** | Precision@5, Recall@5, Faithfulness | Faithfulness > 0.8 |
| **API Performance** | Latency (p95), Throughput | < 500ms, > 100 req/s |
| **System** | Uptime, Error rate | > 99.9%, < 1% |
| **Drift Detection** | Detection rate, False positives | > 90%, < 5% |
| **End-to-End** | Update time, Usability testing | < 2 hours daily |

**Gate condition:** All validation criteria met or explicitly waived for MVP.

**Output:** Validation report, confidence to deploy.

---

## Complete Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              PHASE 0: CONCEPTUAL                                   │
│     01_pitch → 02_investment → 03_statistics → 04_strategy → 05_data → 06_ops     │
│                                                                                     │
│  Output: Confidence that the product idea is sound                                 │
└─────────────────────────────────────┬───────────────────────────────────────────────┘
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           PHASE 1: HIGH-LEVEL DESIGN                               │
│                                                                                     │
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐        │
│  │   MVP Plan          │  │  Product Engineering │  │  Product            │        │
│  │   (Architecture)    │  │  (Scope & Roadmap)   │  │  Expectations       │        │
│  └─────────────────────┘  └─────────────────────┘  └─────────────────────┘        │
│                                                                                     │
│  Key Decisions: MLOps Architecture · Feature Store · SHAP Core · RAG · CI/CD      │
│                                                                                     │
│  Output: System block diagram with clear layer boundaries                           │
└─────────────────────────────────────┬───────────────────────────────────────────────┘
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           PHASE 2: LOW-LEVEL DESIGN                                │
│                                                                                     │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         │
│  │  L1     │ │  L2     │ │  L3     │ │  L4     │ │  L5     │ │  L6     │         │
│  │ Data    │ │ Feature │ │ Feature │ │ ML      │ │ Doc     │ │ Serving │         │
│  │ Ingest  │ │ Store   │ │ Eng     │ │ Engine  │ │ Intel   │ │ API     │         │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘         │
│                                                                                     │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐                                              │
│  │  L7     │ │  L8     │ │  L9     │                                              │
│  │ Monitor │ │ CI/CD   │ │ UI      │                                              │
│  │ & Drift │ │         │ │ Dashboard│                                              │
│  └─────────┘ └─────────┘ └─────────┘                                              │
│                                                                                     │
│  Each LLD: Function signatures · Schemas · Test criteria · Dependencies            │
│                                                                                     │
│  Output: Document an independent engineer can implement from                        │
└─────────────────────────────────────┬───────────────────────────────────────────────┘
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        PHASE 3: PRODUCTION SPECIFICATION                            │
│                                                                                     │
│                      ┌─────────────────────────────┐                               │
│                      │  CapitalQuant Production    │                               │
│                      │  Specification (FROZEN)     │                               │
│                      └─────────────────────────────┘                               │
│                                                                                     │
│  Contains: Unified architecture · Layer specs · Success metrics · Deployment plan  │
│                                                                                     │
│  Output: Single source of truth for all implementation                              │
└─────────────────────────────────────┬───────────────────────────────────────────────┘
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           PHASE 4: PROMPT GENERATION                                │
│                                                                                     │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         │
│  │Prompt L1│ │Prompt L2│ │Prompt L3│ │Prompt L4│ │Prompt L5│ │Prompt L6│         │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘         │
│                                                                                     │
│  Each prompt: Overview · Input/Output specs · Error handling · Test criteria       │
│                                                                                     │
│  Output: Executable prompts for a coding agent (human or LLM)                       │
└─────────────────────────────────────┬───────────────────────────────────────────────┘
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           PHASE 5: IMPLEMENTATION                                   │
│                                                                                     │
│  Implementation Order (Priority-based):                                             │
│                                                                                     │
│  ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐                  │
│  │ L4  │ →  │ L3  │ →  │ L2  │ →  │ L1  │ →  │ L6  │ →  │ L5  │                  │
│  │ML   │    │Feat │    │Store│    │Data │    │API  │    │RAG  │                  │
│  │Engine│    │Eng  │    │     │    │Ingest│   │     │    │     │                  │
│  └─────┘    └─────┘    └─────┘    └─────┘    └─────┘    └─────┘                  │
│                                                                                     │
│  ┌─────┐    ┌─────┐    ┌─────┐                                                    │
│  │ L7  │ →  │ L8  │ →  │ L9  │                                                    │
│  │Monitor│   │CI/CD│    │UI   │                                                    │
│  │&Drift│   │     │    │     │                                                    │
│  └─────┘    └─────┘    └─────┘                                                    │
│                                                                                     │
│  Each layer: Execute prompt → test → commit → tag                                  │
│                                                                                     │
│  Output: Working code, passing tests, deployment artifacts                          │
└─────────────────────────────────────┬───────────────────────────────────────────────┘
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           PHASE 6: VALIDATION                                       │
│                                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │  ML Model    │  │  SHAP        │  │  RAG System  │  │  Production  │           │
│  │  Validation  │  │  Validation  │  │  Validation  │  │  Validation  │           │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘           │
│                                                                                     │
│  Metrics: AUC > 0.55 · Latency < 500ms · Uptime > 99.9% · Faithfulness > 0.8      │
│                                                                                     │
│  Output: Validation report, confidence to deploy                                    │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Current Status

| Phase | Status | Notes |
|-------|--------|-------|
| Phase 0: Conceptual | ✅ Complete | 6 docs finalized |
| Phase 1: HLD | ✅ Complete | Production-grade MLOps architecture defined |
| Phase 2: LLD | ⚠️ In Progress | Layer specifications being detailed |
| Phase 3: Unified Spec | ❌ Not Started | Depends on LLD completion |
| Phase 4: Prompts | ❌ Not Started | Depends on Unified Spec |
| Phase 5: Implementation | ⚠️ Partial | Dashboard mockup exists, no backend yet |
| Phase 6: Validation | ❌ Not Started | Needs data and working model |

---

## Immediate Next Steps

| Priority | Action | Owner | Timeline |
|----------|--------|-------|----------|
| **1** | Define LLD for L4 (ML Engine) | Data Scientist | 3 days |
| **2** | Define LLD for L3 (Feature Engineering) | Data Scientist | 2 days |
| **3** | Define LLD for L6 (Serving API) | Backend Engineer | 2 days |
| **4** | Define LLD for L2 (Feature Store) | Data Engineer | 2 days |
| **5** | Define LLD for L1 (Data Ingestion) | Data Engineer | 2 days |
| **6** | Define LLD for L5 (Document Intelligence) | AI Engineer | 3 days |
| **7** | Define LLD for L7 (Monitoring & Drift) | ML Engineer | 2 days |
| **8** | Define LLD for L8 (CI/CD) | DevOps | 1 day |
| **9** | Define LLD for L9 (UI Dashboard) | Frontend Engineer | 2 days |
| **10** | Generate Unified Spec | Tech Lead | 3 days |

---

## Key Differences from Original SignalIQ Comparison

| Aspect | SignalIQ | CapitalQuant |
|--------|----------|--------------|
| Core Metric | NDI (Normalized Divergence) | Investment Score (XGBoost) |
| Model | Statistical (z-scores) | ML (XGBoost) |
| Explainability | Not central | **SHAP (core)** |
| Documents | Not applicable | **RAG (core)** |
| Target | Price correction | Outperformance vs S&P 500 |
| Validation | KS test, JT test, MWU | AUC-ROC, SHAP stability, RAG Faithfulness |
| **MLOps** | Not specified | **MLflow, Registry, CI/CD, Monitoring, Drift** |
| **Deployment** | Not specified | **Docker, Render, Vercel** |
| **Feature Store** | Not specified | **PostgreSQL + Redis** |
| **Latency Target** | Not specified | **p95 < 500ms** |

