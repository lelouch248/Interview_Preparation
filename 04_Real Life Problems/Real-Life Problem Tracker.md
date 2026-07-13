# Real-Life Problem Tracker

| Problem | Domain | Status | Difficulty | Main Skill | Final Note |
|---|---|---|---|---|---|
| Incremental load without reliable key | ETL | Not Started | Hard | Idempotency |  |
| Snowflake DDL drift detection | Snowflake | In Progress | Hard | Metadata |  |
| Failed task monitoring | Operations | Not Started | Medium | Observability |  |
| Data validation framework | Quality | Not Started | Hard | Framework design |  |

## Problem backlog

- [ ] Incremental load when no trustworthy updated timestamp exists
- [ ] Compare source and target with no primary key
- [ ] Process 50M rows safely in batches
- [ ] Handle partial failure and safe rerun
- [ ] Detect and classify Snowflake object drift
- [ ] Monitor failed Snowflake tasks and alert owners
- [ ] Capture Fivetran/HVR failures in a common dashboard
- [ ] Validate mapping sheets against warehouse metadata
- [ ] Detect schema drift before downstream failure
- [ ] Build source-to-target reconciliation
- [ ] Backfill historical data without breaking daily loads
- [ ] Handle late-arriving facts and dimensions
- [ ] Reduce warehouse cost while protecting SLAs
- [ ] Design data access for multiple teams and environments
- [ ] Build a query-optimization recommendation workflow
- [ ] Build a read-only agentic data-operations control tower
