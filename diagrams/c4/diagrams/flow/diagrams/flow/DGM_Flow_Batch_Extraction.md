
---

# FILE 8 — `diagrams/flow/DGM_Flow_Batch_Extraction.md`

```md
```mermaid
flowchart TD
    A[Batch window allowed?] --> B[Scheduler launches batch extraction]
    B --> C[Fetch records by date/company/domain window]
    C --> D[Chunk into manageable packages]
    D --> E[Publish raw packages to Kafka]
    E --> F[Record batch ID and extraction metrics]
    F --> G[Mark batch as extracted]
