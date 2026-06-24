
---

# FILE 13 — `diagrams/sequence/DGM_SEQ_ReconMismatch.md`

```md
```mermaid
sequenceDiagram
    participant EXT as Extraction Service
    participant FS as FinSight Loader
    participant REC as Reconciliation Service
    participant OPS as Operations Team

    EXT->>REC: Source batch summary (1000 records, INR totals)
    FS->>REC: Target batch summary (998 records, INR totals mismatch)
    REC->>REC: Compare counts, values, referential checks
    REC->>OPS: Raise reconciliation mismatch alert
    OPS->>REC: Review mismatch case
    REC->>REC: Mark exception and retain audit evidence
