---
topic: "Spark & PySpark"
status: Not Started
readiness: L0
tags: [topic-roadmap]
---

# Spark & PySpark

**Priority:** Production Skill

## Outcome

- Understand distributed data processing and Spark execution.
- Write efficient DataFrame transformations.
- Diagnose common skew, shuffle and memory issues.

## Study checklist

### Distributed Foundations

- [ ] Driver, executors, jobs, stages and tasks
- [ ] Partitions and parallelism
- [ ] Lazy evaluation and DAG construction
- [ ] Transformations vs actions
- [ ] Narrow vs wide transformations

### DataFrame API

- [ ] Reading and writing common formats
- [ ] Select, filter, joins and aggregations
- [ ] Window functions
- [ ] UDFs vs built-in functions
- [ ] Handling nulls and malformed data
- [ ] Schema definition and evolution

### Performance

- [ ] Shuffle and partition sizing
- [ ] Broadcast joins
- [ ] Caching and persistence
- [ ] Data skew and salting
- [ ] Predicate pushdown and column pruning
- [ ] Adaptive Query Execution basics

### Reliability and Operations

- [ ] Checkpointing
- [ ] Small files problem
- [ ] Incremental processing
- [ ] Testing PySpark transformations
- [ ] Spark UI and troubleshooting

### Interview Practice

- [ ] Explain a slow Spark job
- [ ] Choose repartition vs coalesce
- [ ] Design a large join with skew
- [ ] Compare Spark, SQL warehouse and pandas

## Evidence of mastery

- [ ] I created useful notes and examples.
- [ ] I solved practical/interview questions independently.
- [ ] I explained the topic aloud without reading.
- [ ] I recorded recurring mistakes in the mistake log.
- [ ] I revisited this topic after a gap and retained it.

## Links

- Notes folder: [[02_Notes/Spark and PySpark/README|Spark & PySpark Notes]]
- Revision: [[06_Revision/Revision Queue]]
- Mistakes: [[06_Revision/Mistake Log]]
