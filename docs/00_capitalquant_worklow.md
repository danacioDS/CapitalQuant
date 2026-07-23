# Product Development Methodology — General Workflow

A methodology for building production-grade ML products, from concept to deployment.

---

## Overview

This methodology flows from **why** (conceptual) → **what** (design) → **how** (implementation) in six distinct phases. It is designed to be **provider-agnostic** and applicable to any ML product, not just financial applications.

---

## Phase 0: Conceptual Foundation

**Purpose:** Establish the product thesis, domain rationale, and strategic positioning before any design or code.

**Questions to answer:**

| # | Document | Question It Answers |
|---|----------|---------------------|
| 01 | Commercial Pitch | Why does this product need to exist? Who pays for it? |
| 02 | Domain Theory | What domain-specific principles justify the approach? |
| 03 | Technical Theory | How do we measure success? Why this approach? |
| 04 | Commercial Strategy | How does this compete? What's the moat? |
| 05 | Data Acquisition | What data sources? Why these sources? |
| 06 | Operational Strategy | How does this scale? |

**Gate condition:** Conceptual docs reviewed and approved by stakeholders.

**Output:** Confidence that the product idea is sound before spending engineering time.

---

## Phase 1: High-Level Design (HLD)

**Purpose:** Translate conceptual foundation into system architecture. Define layers, data flows, and component boundaries without implementation details.

**Documents:**

| Document | Purpose |
|----------|---------|
| **MVP Plan** | Architecture, data flow diagrams, technology choices |
| **Engineering Scope** | Feature scope, MVP boundaries, post-MVP roadmap |
| **Product Expectations** | Output specifications, validation criteria, success metrics |

**Key decisions made here:**
- System architecture (Ingestion → Storage → Features → ML → Serving → Monitoring → UI)
- Core model selection and validation strategy
- Explainability approach (if required)
- Feature store design (offline/online)
- Model registry and lifecycle management
- Deployment strategy (containerization, hosting)
- Monitoring and observability approach
- Drift management strategy
- CI/CD pipeline design

**Gate condition:** HLD reviewed, dependencies identified, resource estimates approved.

**Output:** System block diagram with clear layer boundaries and operational workflow.

---

## Phase 2: Low-Level Design (LLD)

**Purpose:** Detailed technical specifications for each layer. Function signatures, API contracts, database schemas, error handling rules.

**Documents (one per layer):**

| Layer | Focus |
|-------|-------|
| **L1** | Data Ingestion — ETL with validation |
| **L2** | Data Storage / Feature Store — Schemas, point-in-time correctness |
| **L3** | Feature Engineering — Transformations, versioning |
| **L4** | ML Engine — Training, validation, model selection |
| **L5** | Domain Intelligence — RAG, embeddings, specialized processing |
| **L6** | Serving API — Endpoints, async processing, caching |
| **L7** | Monitoring & Drift — Metrics, alerting, drift detection |
| **L8** | CI/CD — Workflows, deployment gates, rollback |
| **L9** | UI / Dashboard — Components, visualizations, user interactions |

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

**What it contains:**

| Section | Content |
|---------|---------|
| **Executive Summary** | One-page overview |
| **Architecture** | Complete diagram with all layers |
| **Layer Specifications** | Synthesized from HLD + LLD |
| **Data Flows** | Training and inference pipelines |
| **Validation Criteria** | Statistical tests, acceptance criteria |
| **Success Metrics** | Model performance, API latency, uptime, domain-specific metrics |
| **Deployment Architecture** | Hosting, scaling, disaster recovery |
| **Monitoring & Alerting** | Configuration, thresholds, response procedures |

**Why this exists:** The unified spec prevents HLD/LLD drift. When HLD and LLD conflict, the unified spec is truth.

**Gate condition:** Unified spec frozen. No changes without formal change request.

**Output:** A single document that drives all implementation prompts.

---

## Phase 4: Prompt Generation

**Purpose:** Convert the unified spec into executable prompts for each module. Each prompt is self-contained and testable.

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
3. Continue through all prompts for the layer
4. Run full layer test suite
5. Verify no regressions in existing layers
6. Tag release (e.g., `v0.2-with-layer1`)

**Recommended implementation order:**

| Priority | Layer | Why This Order |
|----------|-------|----------------|
| **1** | L4: ML Engine | Proves the core thesis works with synthetic data |
| **2** | L3: Feature Engineering | Provides real features to the ML Engine |
| **3** | L2: Data Storage | Stores features for reproducibility |
| **4** | L1: Data Ingestion | Feeds real data into the pipeline |
| **5** | L6: Serving API | Exposes predictions |
| **6** | L5: Domain Intelligence | Adds domain-specific capabilities |
| **7** | L7: Monitoring & Drift | Adds production observability |
| **8** | L8: CI/CD | Automates deployment |
| **9** | L9: UI Dashboard | Presents everything to users |

**Gate condition:** All tests pass, no regressions, synthetic demo works.

**Output:** Working code, passing tests, deployment artifacts.

---

## Phase 6: Validation

**Purpose:** Verify that the implemented system meets the success criteria defined in conceptual and HLD phases.

**Validation activities:**

| Component | Validation Metrics |
|-----------|-------------------|
| **ML Model** | Walk-forward validation, ROC-AUC, Precision, Recall, Accuracy |
| **Explanations** | Global consistency, Local stability, Plausibility |
| **Domain Intelligence** | Precision@5, Recall@5, Faithfulness, Context Precision |
| **API Performance** | Latency (p95), Throughput |
| **System** | Uptime, Error rate |
| **Drift Detection** | Detection rate, False positives |
| **End-to-End** | Update time, Usability testing |

**Gate condition:** All validation criteria met or explicitly waived for MVP.

**Output:** Validation report, confidence to deploy.

---

## Complete Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PHASE 0: CONCEPTUAL                                   │
│  01_pitch → 02_domain → 03_technical → 04_strategy → 05_data → 06_ops    │
│                                                                             │
│  Output: Confidence that the product idea is sound                         │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PHASE 1: HIGH-LEVEL DESIGN                            │
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                     │
│  │  MVP Plan    │  │  Engineering │  │  Product     │                     │
│  │  (Architecture)│  │  Scope      │  │  Expectations│                     │
│  └──────────────┘  └──────────────┘  └──────────────┘                     │
│                                                                             │
│  Key Decisions: Architecture · Data Flows · Technology Stack              │
│                                                                             │
│  Output: System block diagram with clear layer boundaries                  │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PHASE 2: LOW-LEVEL DESIGN                             │
│                                                                             │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐        │
│  │ L1 │ │ L2 │ │ L3 │ │ L4 │ │ L5 │ │ L6 │ │ L7 │ │ L8 │ │ L9 │        │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘        │
│                                                                             │
│  Each LLD: Function signatures · Schemas · Test criteria · Dependencies   │
│                                                                             │
│  Output: Document an independent engineer can implement from               │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PHASE 3: PRODUCTION SPECIFICATION                     │
│                                                                             │
│              ┌─────────────────────────────────────┐                       │
│              │  Production Specification (FROZEN)  │                       │
│              └─────────────────────────────────────┘                       │
│                                                                             │
│  Contains: Unified architecture · Layer specs · Success metrics            │
│                                                                             │
│  Output: Single source of truth for all implementation                     │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PHASE 4: PROMPT GENERATION                            │
│                                                                             │
│  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐             │
│  │Prompt │ │Prompt │ │Prompt │ │Prompt │ │Prompt │ │Prompt │             │
│  │  L1   │ │  L2   │ │  L3   │ │  L4   │ │  L5   │ │  L6   │             │
│  └───────┘ └───────┘ └───────┘ └───────┘ └───────┘ └───────┘             │
│                                                                             │
│  Each prompt: Overview · Input/Output specs · Error handling · Tests      │
│                                                                             │
│  Output: Executable prompts for a coding agent (human or LLM)              │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PHASE 5: IMPLEMENTATION                               │
│                                                                             │
│  Implementation Order:                                                     │
│                                                                             │
│  ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐        │
│  │L4 │ → │L3 │ → │L2 │ → │L1 │ → │L6 │ → │L5 │ → │L7 │ → │L8 │ → │L9 │ │
│  └───┘   └───┘   └───┘   └───┘   └───┘   └───┘   └───┘   └───┘        │
│                                                                             │
│  Each layer: Execute prompt → test → commit → tag                         │
│                                                                             │
│  Output: Working code, passing tests, deployment artifacts                 │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PHASE 6: VALIDATION                                   │
│                                                                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                     │
│  │  Model   │ │  Explain │ │  Domain  │ │  System  │                     │
│  │  Valid.  │ │  Valid.  │ │  Valid.  │ │  Valid.  │                     │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘                     │
│                                                                             │
│  Metrics: Defined in Phase 0 and Phase 1                                   │
│                                                                             │
│  Output: Validation report, confidence to deploy                           │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Key Principles

| Principle | Description |
|-----------|-------------|
| **Concept before Code** | Establish the why before the how |
| **Design before Implementation** | HLD → LLD → Spec before writing code |
| **Testable Units** | Every module has defined test criteria |
| **Frozen Spec** | Once frozen, spec is truth |
| **Incremental Delivery** | Each layer is built, tested, and released |
| **Production from Day 1** | Design for deployment, monitoring, and operations |
| **Domain-Agnostic** | Applicable to any ML product |

---

## Governance

| Gate | Condition |
|------|-----------|
| **Phase 0 → 1** | Conceptual docs approved |
| **Phase 1 → 2** | HLD reviewed, dependencies identified |
| **Phase 2 → 3** | LLD complete, no ambiguities |
| **Phase 3 → 4** | Spec frozen |
| **Phase 4 → 5** | Prompts reviewed and testable |
| **Phase 5 → 6** | All tests pass, no regressions |
| **Phase 6 → Deploy** | All validation criteria met |

---

## Success Metrics (Generic)

| Metric Type | Examples |
|-------------|----------|
| **Model Performance** | AUC, Precision, Recall, F1, Calibration |
| **System Performance** | Latency (p95), Throughput, Uptime |
| **Data Quality** | Completeness, Accuracy, Freshness |
| **Operational** | Drift detection rate, Retraining success rate |
| **Business** | User adoption, Time saved, Decision quality |

---

*This methodology provides a structured, repeatable approach to building production-grade ML products from concept to deployment.*