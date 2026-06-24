
---

# FILE 12 — `diagrams/sequence/DGM_SEQ_ErrorRetry.md`

```md
```mermaid
sequenceDiagram
    participant TR as Transform Service
    participant FS as FinSight Loader
    participant FIN as FinSight
    participant RET as Retry Handler
    participant DLQ as Dead Letter Queue

    TR->>FS: Send transformed payload batch
    FS->>FIN: POST batch
    FIN-->>FS: HTTP 503 Service Unavailable
    FS->>RET: Register transient failure
    RET->>RET: Calculate backoff with jitter
    RET->>FS: Retry batch
    FS->>FIN: POST batch retry
    alt Retry succeeds
        FIN-->>FS: 200 Accepted
    else Retry exhausted
        FIN-->>FS: 503 / timeout
        FS->>DLQ: Store failed payload + metadata
    end
