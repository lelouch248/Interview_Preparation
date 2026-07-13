---
topic: "ETL, ELT & Orchestration"
status: Not Started
readiness: L0
tags: [topic-roadmap]
---

# ETL, ELT & Orchestration

**Priority:** Core

## Outcome

- Design resilient batch and incremental pipelines.
- Handle retries, backfills, late data and schema changes.
- Explain pipeline operations from source to serving layer.

## Study checklist

### Pipeline Foundations

- [ ] ETL vs ELT
- [ ] Batch vs streaming
- [ ] Full refresh vs incremental load
- [ ] Push vs pull ingestion
- [ ] Control tables and metadata-driven pipelines

### Incremental Processing

- [ ] High-water marks and timestamps
- [ ] CDC logs and change tables
- [ ] Hash comparison and diff strategies
- [ ] Idempotency and exactly-once effects
- [ ] Deletes, replays and backfills
- [ ] Handling missing primary keys

### Orchestration

- [ ] DAG design and dependency management
- [ ] Scheduling, SLAs and time zones
- [ ] Retries, exponential backoff and dead-letter handling
- [ ] Parameters, environments and secrets
- [ ] Backfill and rerun design
- [ ] Event-driven orchestration

### Reliability

- [ ] Checkpointing and restartability
- [ ] Schema evolution and contract checks
- [ ] Data reconciliation and completeness checks
- [ ] Alerting and incident ownership
- [ ] Runbooks and operational metadata

### Interview Scenarios

- [ ] 50M-row migration with limited downtime
- [ ] Daily source without reliable update timestamp
- [ ] Late-arriving data and duplicate events
- [ ] Partial pipeline failure and safe rerun
- [ ] Cross-region source and warehouse design

## Evidence of mastery

- [ ] I created useful notes and examples.
- [ ] I solved practical/interview questions independently.
- [ ] I explained the topic aloud without reading.
- [ ] I recorded recurring mistakes in the mistake log.
- [ ] I revisited this topic after a gap and retained it.

## Links

- Notes folder: [[02_Notes/ETL ELT and Orchestration/README|ETL, ELT & Orchestration Notes]]
- Revision: [[06_Revision/Revision Queue]]
- Mistakes: [[06_Revision/Mistake Log]]
