
---

# FILE 9 — `diagrams/flow/DGM_Flow_Error_Retry.md`

```md
```mermaid
flowchart TD
    A[FinSight load attempt] --> B{Success?}
    B -- Yes --> C[Mark record delivered]
    B -- No --> D{Transient error?}
    D -- Yes --> E[Apply backoff + retry policy]
    E --> F{Retry attempts exhausted?}
    F -- No --> A
    F -- Yes --> G[Route to DLQ with error metadata]
    D -- No --> H[Route to DLQ / business exception queue]
