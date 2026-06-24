# D1 – Integration Architecture Document
## SAP S/4HANA to Zetheta FinSight Financial Integration
**Client:** Meridian Manufacturing Ltd.  
**Author:** Anshika Singh  
**Version:** 1.0

---

# 1. Executive Summary
Meridian Manufacturing Ltd. operates SAP S/4HANA as its system of record for finance and operations, while Zetheta FinSight serves as the analytics platform for financial dashboards, reporting, and executive decision support. The current state requires a reliable integration capability that can move data from SAP into FinSight with strong guarantees of correctness, auditability, operational resilience, and compliance.

This document proposes a **hybrid SAP-to-FinSight integration architecture** built around:
- **ODP/CDS-based delta extraction** for high-priority and frequently changing financial/master data
- **scheduled batch extraction** for larger-volume, lower-frequency, or historical domains
- a **Kafka-backed integration layer** for decoupling and scalability
- a **transformation and validation layer** for schema conversion, enrichment, and rule enforcement
- **idempotent FinSight load APIs**
- a **reconciliation service** for completeness, accuracy, consistency, and timeliness checks
- a full **observability and operational control framework** including logs, metrics, alerts, retry management, and dead-letter handling

The design explicitly respects Meridian’s real-world constraints, including SAP batch windows, ODP extraction limits, RFC pool restrictions, network bandwidth controls, Indian data residency, and GST-sensitive financial transformation requirements.

---

# 2. Business Context

## 2.1 Business Problem
Meridian Manufacturing requires a dependable financial integration bridge between SAP S/4HANA and FinSight so that finance leadership, audit, and operations stakeholders can access timely and trusted analytical views of:
- General Ledger
- Accounts Payable
- Accounts Receivable
- Cost Centre and Profit Centre reporting
- Material cost and inventory valuation
- Purchase and sales commitments
- Fixed asset financials
- Bank statement and cash visibility
- Budget vs actual variance reporting

Today, a lack of unified, automated integration would create:
- stale or inconsistent reporting
- manual reconciliation overhead
- limited audit traceability
- delayed visibility into cost and profitability
- higher risk during close and compliance reporting cycles

## 2.2 Integration Objectives
The integration solution must:
1. extract source data from SAP safely and efficiently
2. transform SAP structures into FinSight-compatible schemas
3. support both delta and batch patterns
4. guarantee delivery reliability with retries, idempotency, and failure isolation
5. provide mathematical and operational reconciliation
6. maintain complete lineage and audit traceability
7. support monitoring, support handover, and controlled deployment

---

# 3. Client Environment Summary

## 3.1 Source and Target Landscape
| Layer | System | Role |
|---|---|---|
| ERP | SAP S/4HANA 2023 FPS02 | Source of financial and operational data |
| Analytics | Zetheta FinSight 4.2 | Destination analytics/reporting platform |
| Warehouse | Snowflake | Historical and analytical persistence |
| Identity | Azure Active Directory | Authentication / authorisation |
| Monitoring | Nagios + Grafana | Existing monitoring ecosystem |
| Connectivity | MPLS + SD-WAN | SAP DC to AWS Mumbai connectivity |

## 3.2 Manufacturing Footprint
Meridian operates multiple plants mapped to SAP company codes MC01, MC02, and MC03, which implies:
- multi-company-code processing
- plant-to-company routing awareness
- regional cost/profit visibility
- possible different operational data volumes by plant

---

# 4. Scope of Integration

## 4.1 In-Scope Data Domains
The architecture is designed for the following 12 source domains:
1. General Ledger
2. Accounts Payable
3. Accounts Receivable
4. Cost Centre Accounting
5. Profit Centre Accounting
6. Material Ledger
7. Purchase Orders
8. Sales Orders
9. Fixed Assets
10. Bank Statements
11. Budget vs Actual
12. Inventory

## 4.2 SAP Source Reference Tables
The architecture and future mappings are based on relevant SAP tables including:
- **ACDOCA, BKPF, BSEG** – General Ledger
- **BSIK, LFA1, LFC1** – Accounts Payable / vendor
- **BSID, KNA1, KNC1** – Accounts Receivable / customer
- **CSKS, CSKT** – Cost Centre
- **CEPC, CEPCT** – Profit Centre
- **MBEW, MARD** – Material valuation / inventory
- **EKKO, EKPO, EKET** – Purchase Orders
- **VBAK, VBAP, VBEP** – Sales Orders
- **ANLA, ANLZ, ANLP** – Fixed Assets
- **FEBKO, FEBEP** – Bank Statements
- **TCURR** – Exchange rates
- **SETNODE / SETLEAF** – hierarchy structures

---

# 5. Requirements

## 5.1 Functional Requirements
FR-01. Extract SAP financial and master data for all in-scope domains  
FR-02. Support incremental delta extraction for frequently changing datasets  
FR-03. Support scheduled batch extraction for large or lower-priority datasets  
FR-04. Transform SAP source structures into FinSight-compatible target payloads  
FR-05. Validate data for completeness, format, business rule, and referential integrity  
FR-06. Load transformed records into FinSight via authenticated APIs  
FR-07. Prevent duplicate loads through idempotent processing and business keys  
FR-08. Reconcile source and target at batch, document, and amount levels  
FR-09. Preserve GST-sensitive tax amounts and financial precision  
FR-10. Produce operational logs, metrics, alerts, and audit trail artifacts  
FR-11. Support failure isolation, retry, DLQ routing, and reprocessing  
FR-12. Provide stakeholder-ready operational and business documentation

## 5.2 Non-Functional Requirements
NFR-01. Minimise SAP production impact  
NFR-02. Scale to Meridian’s monthly transaction volume with headroom for seasonal spikes  
NFR-03. Maintain traceability from SAP business document to FinSight record  
NFR-04. Support data freshness targets by domain  
NFR-05. Keep all financial data processing within India-hosted infrastructure  
NFR-06. Secure data in transit and at rest  
NFR-07. Provide high observability and actionable alerting  
NFR-08. Allow controlled deployment, rollback, and support handover

---

# 6. Business and Technical Constraints

## 6.1 SAP / Platform Constraints
- No heavy extraction during **01:00–04:30 IST** SAP batch window
- ODP delta extraction restricted to **once every 30 minutes per provider**
- Maximum **50 concurrent RFC connections**
- Shared network bandwidth between Pune data centre and AWS Mumbai; integration must remain within business-hours bandwidth tolerance
- SAP maintenance every 2nd and 4th Saturday 22:00–06:00 IST
- FinSight maintenance first Sunday of month 02:00–06:00 IST

## 6.2 Compliance / Audit Constraints
- Financial data must remain **within Indian borders**
- GST amount structure (CGST/SGST/IGST) must not be corrupted by transformation
- audit requires complete lineage from SAP source document to FinSight analytical record

---

# 7. Architecture Principles

1. **Hybrid extraction strategy** – use delta where freshness matters and batch where efficiency matters.
2. **Loose coupling through messaging** – extraction must be decoupled from downstream transformation and load.
3. **Schema-first integration** – APIs and mappings are explicit, versioned, and testable.
4. **Idempotent target writes** – retries must not create duplicates.
5. **Reconciliation-first finance design** – financial integrations must prove completeness and accuracy.
6. **Operational resilience** – transient failure must be recoverable without manual intervention where possible.
7. **Observability by default** – every batch and every error must be visible.
8. **Compliance-aware transformation** – tax, currency, period, and hierarchy semantics must be preserved.

---

# 8. Proposed Solution Overview

## 8.1 Architecture Summary
The proposed integration architecture consists of six major layers:

1. **SAP Source Layer**
   - SAP S/4HANA tables, CDS views, ODP providers, and selective extraction interfaces

2. **Extraction Layer**
   - scheduled SAP extractor
   - delta token / watermark management
   - package management and throttling
   - SAP-safe extraction orchestration

3. **Messaging & Orchestration Layer**
   - Kafka topics by domain
   - scheduler/orchestrator
   - retry topic and DLQ topic patterns

4. **Transformation & Validation Layer**
   - domain-specific transformation logic
   - enrichment, validation, normalisation, business-rule checks
   - mapping and schema conversion

5. **Target Load Layer**
   - FinSight API client / loader
   - OAuth token handling
   - rate limit management
   - idempotent upsert behaviour

6. **Control, Audit & Observability Layer**
   - reconciliation service
   - structured logs
   - monitoring metrics
   - alerting rules
   - lineage and audit records

---

# 9. Detailed Architecture

## 9.1 Source Extraction Strategy

### 9.1.1 Delta Extraction (Preferred for frequently changing data)
Delta extraction is proposed for:
- General Ledger
- AP open items
- AR open items
- Cost centre / profit centre master updates
- Material Ledger deltas
- bank statement line additions
- selected PO / SO status changes

**Mechanism:** SAP CDS + ODP subscriptions with delta token tracking

**Why this approach**
- efficient incremental loads
- reduced bandwidth
- lower source system impact
- recoverable extraction state
- good fit for near-real-time / periodic analytics sync

### 9.1.2 Scheduled Batch Extraction
Batch extraction is proposed for:
- historical loads
- budget/actual snapshots
- inventory snapshots
- large fixed-asset periodic values
- lower-frequency hierarchy extracts
- fallback recovery loads

**Mechanism:** scheduled extraction jobs using approved SAP interfaces / views with windowed filters

### 9.1.3 Watermark / Token Strategy
Each source domain maintains:
- `last_successful_delta_token`
- `last_successful_batch_timestamp`
- `last_reconciled_batch_id`
- extraction status and record counts

This allows safe retry without skipping data.

---

## 9.2 Integration Platform Containers

### 9.2.1 API Gateway / Ingress Layer
Provides:
- secure entry for admin/control APIs
- rate limiting and policy enforcement
- standard logging and correlation IDs
- future extensibility for external admin or reprocessing endpoints

### 9.2.2 Scheduler / Orchestrator
Responsible for:
- launching extraction jobs by domain and cadence
- respecting SAP batch windows and maintenance windows
- limiting concurrency
- managing dependency ordering (e.g., master data before transactional reconciliation checks)
- triggering backfill or replay jobs

### 9.2.3 SAP Extraction Service
Responsibilities:
- connect to SAP via approved extraction patterns
- request delta/batch data
- chunk large payloads
- attach metadata such as source batch ID, extraction timestamp, and company code
- publish raw events to Kafka

### 9.2.4 Kafka Messaging Backbone
Kafka topics decouple source extraction from transformation and target load.

Suggested topics:
- `sap.gl.raw`
- `sap.ap.raw`
- `sap.ar.raw`
- `sap.costcentre.raw`
- `sap.profitcentre.raw`
- `sap.material.raw`
- `sap.po.raw`
- `sap.so.raw`
- `sap.fixedasset.raw`
- `sap.bank.raw`
- `sap.budget.raw`
- `sap.inventory.raw`

Additional control topics:
- `integration.retry`
- `integration.dlq`
- `integration.reconciliation.events`

### 9.2.5 Transformation & Validation Service
Responsibilities:
- convert SAP source fields into FinSight payload schemas
- format dates, currencies, keys, and hierarchy structures
- apply business rules and validation
- enrich with reference/master data where required
- split valid vs invalid records
- route bad records to business exception queue or DLQ as appropriate

### 9.2.6 FinSight Loader Service
Responsibilities:
- call FinSight APIs
- obtain and refresh OAuth tokens
- enforce idempotency keys
- handle API rate limits and backoff
- persist delivery status
- publish success/failure events

### 9.2.7 Reconciliation Service
Responsibilities:
- compare source and target counts
- compare financial totals
- detect referential breaks
- publish reconciliation reports
- support daily and monthly audit reporting

### 9.2.8 Monitoring / Logging / Alerting
Responsibilities:
- capture structured logs with correlation IDs
- expose metrics to Prometheus/Grafana
- route severe incidents to PagerDuty / support channels
- surface stale pipelines, retry spikes, and reconciliation failures

---

# 10. Target Data Loading Strategy

## 10.1 FinSight Load Pattern
Transformed records are loaded into FinSight through domain-specific REST endpoints using:
- OAuth 2.0 bearer tokens
- domain batch payloads
- idempotency keys derived from business identifiers + batch metadata
- bounded retries for transient failures

## 10.2 Idempotency Strategy
Each target record is written with a deterministic key based on domain semantics, e.g.:
- GL: `companyCode-fiscalYear-documentNumber-lineItem`
- AP open item: `vendor-company-document-clearingStatus`
- Cost centre master: `controllingArea-costCentre-validFrom`
- PO item: `purchaseOrder-item-scheduleLine`

If a retry sends the same logical record again, the target upsert must not create a duplicate.

---

# 11. Reconciliation by Design

The architecture includes reconciliation as a first-class control layer, not a post-processing afterthought.

## 11.1 Reconciliation Dimensions
1. **Completeness** – all source records expected in a batch reached the target
2. **Accuracy** – values match after transformation and rounding rules
3. **Consistency** – references resolve across domains (e.g. GL to cost centre)
4. **Timeliness** – batch completes within SLA

## 11.2 Reconciliation Levels
- batch-level count reconciliation
- financial amount reconciliation
- document-level presence reconciliation
- referential integrity reconciliation
- SLA / freshness reconciliation

---

# 12. Error Handling Design Principles

Although the full framework is specified in D4, the architecture embeds the following patterns:
- transient vs permanent vs data-quality vs business-rule vs system error separation
- bounded retries with backoff
- retry topics and delayed retry scheduling
- DLQ with payload, metadata, and error context
- circuit breaker for repeated target API failure
- replay tooling for manual remediation

---

# 13. Security Architecture

## 13.1 Authentication
- FinSight API access via OAuth 2.0 access tokens
- token refresh flow handled by loader service
- least-privilege scopes by domain or write capability

## 13.2 Authorisation
- service accounts for extraction and target load
- environment-separated credentials
- role-based access to operational tools and reprocessing functions

## 13.3 Data Protection
- TLS 1.2+ for all in-transit communication
- encrypted storage for message retention, audit logs, and DLQ payloads
- no financial processing outside India-hosted infrastructure
- secrets managed outside codebase

---

# 14. Observability Architecture

## 14.1 Logs
All services emit structured JSON logs including:
- timestamp
- service name
- environment
- correlation_id
- batch_id
- domain
- operation
- severity
- outcome
- error_code
- retry_count
- source/target reference

## 14.2 Metrics
Key metrics include:
- records extracted per domain
- records transformed / rejected
- records loaded
- API latency and error rates
- retry volume
- DLQ depth
- reconciliation mismatch count
- data freshness / SLA compliance

## 14.3 Alerts
High-priority alert categories include:
- pipeline stale > SLA
- repeated FinSight API failures
- reconciliation mismatch
- high DLQ growth
- token/authentication failure
- SAP extraction failures across multiple domains

---

# 15. Network Architecture

## 15.1 Connectivity Model
SAP remains in Meridian’s on-premise / client-controlled environment. The integration platform and FinSight operate within India-based cloud infrastructure (AWS Mumbai or equivalent India region). Connectivity between SAP and cloud services is established through Meridian’s MPLS + SD-WAN path with controlled bandwidth consumption and secure transport.

## 15.2 Network Design Goals
- isolate SAP production from direct external load
- minimise data movement overhead
- secure all cross-boundary communication
- provide observability and operational access without exposing source systems unnecessarily

---

# 16. Deployment Architecture

## 16.1 Proposed Runtime Layout
- **Containerised integration services** deployed on Kubernetes / managed container runtime in India region
- **Kafka cluster** for durable event transport
- **PostgreSQL / metadata store** for watermark, job status, reconciliation summaries, and configuration
- **Object storage** for audit reports, reconciliation outputs, and archived artifacts
- **Prometheus + Grafana + ELK-compatible sinks** for observability

## 16.2 Service Breakdown
- extractor service
- scheduler service
- transformation service
- validation/rules service
- FinSight loader service
- reconciliation service
- operational admin / replay utility

---

# 17. Technology Stack Selection

## 17.1 Selected Stack
| Layer | Selected Technology | Rationale |
|---|---|---|
| Architecture documentation | Markdown + Draw.io + Mermaid | version control friendly and fast iteration |
| API specification | OpenAPI 3.0 YAML + Swagger Editor + Spectral | industry-standard, lintable |
| Messaging backbone | Kafka | scalable event decoupling and replay capability |
| Orchestration | Scheduler + cron / workflow orchestration pattern | controlled timed extraction and replay |
| Integration services | containerised stateless services | deployment flexibility and horizontal scaling |
| Monitoring | Prometheus + Grafana + ELK-compatible logs | aligns with project expectations and enterprise patterns |
| Metadata store | relational store for watermark/config/reconciliation | operational state management |
| Identity | Azure AD + OAuth integration | consistent with client landscape |

## 17.2 Alternatives Considered
### Alternative A – Direct SAP to FinSight synchronous API calls
Rejected because:
- tightly coupled
- poor retry isolation
- high SAP impact risk
- difficult to scale and replay
- weak observability

### Alternative B – Batch-only nightly ETL
Rejected because:
- insufficient freshness for CFO analytics use case
- weaker operational responsiveness
- delayed reconciliation visibility

### Alternative C – Fully real-time event-only architecture
Rejected because:
- not all domains need real-time
- ODP and SAP production constraints make full real-time unnecessary and expensive
- batch remains better for some historical or periodic domains

---

# 18. Data Freshness Strategy by Domain

| Domain | Pattern | Proposed Cadence |
|---|---|---|
| General Ledger | Delta | every 30 minutes |
| Accounts Payable | Delta | every 30 minutes |
| Accounts Receivable | Delta | every 30 minutes |
| Cost Centre | Delta + daily hierarchy sync | every 4 hours / daily full hierarchy |
| Profit Centre | Delta + daily hierarchy sync | every 4 hours / daily full hierarchy |
| Material Ledger | Delta + scheduled batch | hourly / daily valuation batch |
| Purchase Orders | Delta | every 30–60 minutes |
| Sales Orders | Delta | every 30–60 minutes |
| Fixed Assets | batch + periodic delta | daily |
| Bank Statements | batch / event-assisted | hourly or bank statement arrival-based |
| Budget vs Actual | batch | daily |
| Inventory | snapshot batch | every 4 hours or end-of-day depending business need |

---

# 19. Assumptions
1. SAP extraction interfaces required for all in-scope domains are available or can be represented via CDS/ODP or approved extraction views.
2. FinSight exposes domain endpoints for master and transactional financial data.
3. Meridian approves an India-region integration deployment footprint.
4. Reference/master data required for validation is available within extraction windows.
5. Company code to analytics tenant mapping is defined.
6. GST-relevant source fields are extractable in required domains.
7. FinSight supports idempotent write semantics or equivalent deduplication logic.

---

# 20. Risks and Mitigations (summary)
A detailed risk register is maintained separately, but key risks include:
- SAP extraction performance impact
- schema evolution in SAP source views
- FinSight API rate limiting / throttling
- GST master data quality issues
- reconciliation mismatches due to master data lag
- network instability between SAP and cloud
- operational alert fatigue if thresholds are poorly tuned

---

# 21. Architecture Decision Summary
The recommended architecture for Meridian Manufacturing is a **hybrid event-assisted batch integration platform** with:
- **ODP/CDS delta extraction** for freshness-sensitive domains
- **scheduled batch extraction** for larger or lower-priority domains
- **Kafka-backed decoupling**
- **transformation and validation services**
- **idempotent FinSight load APIs**
- **reconciliation and observability as first-class services**

This design is the best fit because it balances:
- SAP safety
- operational reliability
- data freshness
- auditability
- compliance
- scalability
- practical enterprise implementation constraints

---

# 22. Deliverables Produced as Part of D1
The D1 architecture package includes:
- system context diagram
- container diagram
- component diagram
- network architecture diagram
- deployment architecture diagram
- four data flow diagrams
- three sequence diagrams
- risk register
- non-functional requirements summary
- technology stack justification
