# Problem — Incremental Load Without a Reliable Key

## Scenario

A large source table has no dependable primary key and no trustworthy update timestamp. You need a repeatable incremental process into Snowflake.

## Questions to solve

- How will you identify new, changed and deleted rows?
- What columns can form a stable business fingerprint?
- Where will current and previous hashes be stored?
- How will the process handle duplicates and collisions?
- How will you batch 50M+ rows?
- How will a failed run restart safely?
- What reconciliation proves completeness?
- When is a periodic full comparison acceptable?

Use [[99_Templates/Real-Life Problem Template]].
