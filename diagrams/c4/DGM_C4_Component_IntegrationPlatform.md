
---

# FILE 6 — `diagrams/c4/DGM_C4_Component_IntegrationPlatform.md`

```md
```mermaid
flowchart TB
    subgraph IntegrationPlatform
        O1[Scheduler]
        O2[Job Controller]
        E1[ODP Delta Extractor]
        E2[Batch Extractor]
        E3[Watermark Manager]
        M1[Kafka Producer]
        T1[Schema Mapper]
        T2[Validation Engine]
        T3[Business Rules Engine]
        T4[Enrichment Module]
        L1[OAuth Client]
        L2[FinSight API Client]
        L3[Idempotency Manager]
        R1[Reconciliation Calculator]
        R2[Exception Manager]
        X1[Retry Handler]
        X2[DLQ Router]
        OBS[Observability Adapter]
    end

    O1 --> O2
    O2 --> E1
    O2 --> E2
    E1 --> E3
    E2 --> E3
    E1 --> M1
    E2 --> M1
    M1 --> T1
    T1 --> T2
    T2 --> T3
    T3 --> T4
    T4 --> L2
    L1 --> L2
    L2 --> L3
    L3 --> R1
    T2 --> X2
    L2 --> X1
    X1 --> X2
    R1 --> R2
    E1 --> OBS
    E2 --> OBS
    T2 --> OBS
    L2 --> OBS
    R1 --> OBS
