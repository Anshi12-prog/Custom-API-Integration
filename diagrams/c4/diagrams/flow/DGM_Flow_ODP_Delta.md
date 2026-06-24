
---

# FILE 7 — `diagrams/flow/DGM_Flow_ODP_Delta.md`

```md
```mermaid
flowchart TD
    A[Scheduler triggers domain extraction] --> B[Read last successful delta token]
    B --> C[Request ODP delta from SAP CDS/Provider]
    C --> D[Receive changed records package]
    D --> E[Attach batch metadata + correlation ID]
    E --> F[Publish raw records to Kafka topic]
    F --> G[Persist extraction success metadata]
    G --> H[Advance delta token only after successful extraction persistence]
