---
topic: "Fivetran & HVR"
status: Not Started
readiness: L0
tags: [topic-roadmap]
---

# Fivetran & HVR

**Priority:** Production Skill

## Outcome

- Configure and troubleshoot managed ingestion.
- Understand CDC behavior, schema changes and sync history.
- Operate connectors with clear monitoring and recovery steps.

## Study checklist

### Connector Foundations

- [ ] Sources, destinations and connector setup
- [ ] Initial sync vs incremental sync
- [ ] Schema mapping and naming
- [ ] Connection tests and permissions
- [ ] Sync frequency and scheduling

### Change Capture

- [ ] Log-based CDC concepts
- [ ] History Mode
- [ ] Deletes and soft deletes
- [ ] Re-sync behavior
- [ ] Primary-key and no-primary-key implications
- [ ] HVR capture and integrate concepts

### Operations

- [ ] Connector logs and error classification
- [ ] Schema drift and column changes
- [ ] Failed sync recovery
- [ ] Network and firewall issues
- [ ] Destination permissions and warehouse impact
- [ ] Usage and cost awareness

### Transformations and Governance

- [ ] dbt transformations after load
- [ ] Environment separation
- [ ] Sensitive-column handling
- [ ] Audit evidence and ownership
- [ ] Runbooks and alerting

### Interview Practice

- [ ] Diagnose a stalled connector
- [ ] Plan a zero/low-downtime re-sync
- [ ] Handle source schema changes
- [ ] Compare Fivetran with custom ingestion

## Evidence of mastery

- [ ] I created useful notes and examples.
- [ ] I solved practical/interview questions independently.
- [ ] I explained the topic aloud without reading.
- [ ] I recorded recurring mistakes in the mistake log.
- [ ] I revisited this topic after a gap and retained it.

## Links

- Notes folder: [[02_Notes/Fivetran and HVR/README|Fivetran & HVR Notes]]
- Revision: [[06_Revision/Revision Queue]]
- Mistakes: [[06_Revision/Mistake Log]]
