
---

# 6) RISK REGISTER STARTER
Add this section either at the bottom of D1 or in a separate `docs/Risk_Register.md`.

```md
# Risk Register

| Risk ID | Risk Description | Probability | Impact | Mitigation |
|---|---|---:|---:|---|
| R-01 | SAP extraction impacts production performance during peak periods | Medium | High | Restrict extraction windows, throttle package sizes, prefer ODP deltas |
| R-02 | ODP provider schema changes break mappings | Medium | High | Version API/mappings, add schema validation and monitoring |
| R-03 | FinSight API rate limiting delays data freshness | Medium | Medium | Batch intelligently, implement backoff, prioritise critical domains |
| R-04 | GST-related source data quality issues cause invalid downstream records | High | High | Add GST validation rules, quarantine bad records, notify AP/AR owners |
| R-05 | RFC pool exhaustion affects SAP dialog users | Low | High | Cap concurrent extractions and avoid unnecessary RFC-based patterns |
| R-06 | Network instability between SAP and cloud causes partial batches | Medium | Medium | Retry, chunk payloads, persist extraction state and use replay |
| R-07 | Reconciliation mismatches due to master data timing gaps | Medium | High | Sequence master data loads, add referential checks and reconciliation exceptions |
| R-08 | Audit trail gaps prevent document-level lineage | Low | High | Enforce correlation IDs, persist batch/document lineage metadata |
| R-09 | Over-alerting causes operational fatigue | Medium | Medium | Define severity thresholds and suppression logic |
| R-10 | India data residency requirements are accidentally violated by tooling/configuration | Low | High | Restrict processing/storage to India region, document hosting boundaries |
