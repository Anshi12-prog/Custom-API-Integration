 
---

# FILE 5 — `diagrams/c4/DGM_C4_Container.md`

```md
```mermaid
flowchart LR
    SAP[SAP S/4HANA]
    EXT[SAP Extraction Service]
    SCH[Scheduler / Orchestrator]
    META[Metadata & Watermark Store]
    KAFKA[Kafka Event Backbone]
    TR[Transformation & Validation Service]
    LOAD[FinSight Loader Service]
    REC[Reconciliation Service]
    DLQ[Retry + Dead Letter Queue]
    MON[Monitoring / Logging / Alerting]
    FIN[FinSight]
    SNOW[Snowflake]

    SCH --> EXT
    META --> EXT
    SAP --> EXT
    EXT --> KAFKA
    KAFKA --> TR
    TR --> LOAD
    LOAD --> FIN
    LOAD --> SNOW
    TR --> DLQ
    LOAD --> DLQ
    EXT --> MON
    TR --> MON
    LOAD --> MON
    REC --> MON
    EXT --> REC
    LOAD --> REC
