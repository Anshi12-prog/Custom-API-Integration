
---

# FILE 11 — `diagrams/sequence/DGM_SEQ_HappyPath.md`

```md
```mermaid
sequenceDiagram
    participant SCH as Scheduler
    participant SAP as SAP S/4HANA
    participant EXT as Extraction Service
    participant K as Kafka
    participant TR as Transform Service
    participant FS as FinSight Loader
    participant FIN as FinSight
    participant REC as Reconciliation Service

    SCH->>EXT: Trigger GL delta extraction
    EXT->>SAP: Request changed journal entries since delta token
    SAP-->>EXT: Return delta package
    EXT->>K: Publish raw journal entry events
    K->>TR: Deliver raw events
    TR->>TR: Transform + validate + enrich
    TR->>FS: Send FinSight payload
    FS->>FIN: POST /journal-entries
    FIN-->>FS: 200 Accepted
    FS->>REC: Send batch delivery summary
    EXT->>REC: Send source extraction summary
    REC->>REC: Reconcile counts + amounts
