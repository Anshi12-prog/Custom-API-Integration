
---

# FILE 10 — `diagrams/flow/DGM_Flow_Reconciliation_Audit.md`

```md
```mermaid
flowchart TD
    A[Source batch extracted] --> B[Target batch delivered]
    B --> C[Reconciliation service receives source + target counts]
    C --> D[Run completeness checks]
    D --> E[Run amount / value checks]
    E --> F[Run referential integrity checks]
    F --> G{All checks pass?}
    G -- Yes --> H[Mark batch reconciled]
    G -- No --> I[Raise reconciliation exception]
    I --> J[Notify operations + audit trail entry]
