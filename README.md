# 40 Days of Software Engineering Skills Improvement

Welcome to my repository dedicated to improving my software engineering skills over 40 days. This repo contains daily plans, exercises, and resources to help me level up my knowledge and practical abilities. Currently, I’ve detailed the schedule for Days 1–10. In the coming weeks, I’ll add the remaining days’ schedules.

## Table of Contents
- [Day 1: Database Fundamentals & PostgreSQL Setup](#day-1-database-fundamentals--postgresql-setup)
- [Day 2: Advanced Database Design Principles & ER Diagrams](#day-2-advanced-database-design-principles--er-diagrams)
- [Day 3: Indexing Strategies](#day-3-indexing-strategies)
- [Day 4: Query Optimization](#day-4-query-optimization)
- [Day 5: Practice Scenario – Healthcare Data Schema](#day-5-practice-scenario--healthcare-data-schema)
- [Day 6: ORM Usage Patterns](#day-6-orm-usage-patterns)
- [Day 7: Caching Strategies](#day-7-caching-strategies)
- [Day 8: Database Migration Strategies](#day-8-database-migration-strategies)
- [Day 9: High Availability, Backup & Recovery](#day-9-high-availability-backup--recovery)
- [Day 10: Practice Scenario – Financial Transaction System & Security Deep Dive](#day-10-practice-scenario--financial-transaction-system--security-deep-dive)

---

## Day 1: Database Fundamentals & PostgreSQL Setup

### Theory (40 min)
- **Overview of PostgreSQL architecture:** storage engine, transaction management, and concurrency control.
- **Understanding transaction isolation levels and ACID properties.**

### Practical (30 min)
- Set up PostgreSQL in Docker with persistent volumes.
- Run basic SQL commands and verify connectivity.

### Resources
- PostgreSQL documentation (chapters 1–3)
- Docker volumes documentation

---

## Day 2: Advanced Database Design Principles & ER Diagrams

### Theory (40 min)
- Deep dive into normalization (1NF through BCNF).
- Discussion on composite keys vs. surrogate keys and how constraints serve as documentation.

### Practical (30 min)
- Evaluate a sample, flawed schema and identify at least three key improvements.
- **Design Exercise:** Use [dbdiagram.io](https://dbdiagram.io) to create and refine ER diagrams for the improved schema.

### Focus
- Applying design principles and visualizing relationships to support clarity and maintenance.

---

## Day 3: Indexing Strategies

### Theory (40 min)
- Explore various index types (B-tree, GIN, covering indexes) and their use cases.
- Learn when to create specialized indexes to optimize query performance.

### Practical (30 min)
- Create different indexes on a sample dataset.
- Use `EXPLAIN ANALYZE` to compare query plans and measure performance gains.

### Tools
- PostgreSQL’s indexing and query plan features

---

## Day 4: Query Optimization

### Theory (40 min)
- Understand how the PostgreSQL query planner/optimizer works, and the role of statistics and cost estimation.
- Identify common performance bottlenecks and strategies to overcome them.

### Practical (30 min)
- Optimize three progressively complex queries.
- Practice reading and interpreting query plans to identify inefficient operations.

### Focus
- Rewriting queries and tweaking database statistics for improved efficiency.

---

## Day 5: Practice Scenario – Healthcare Data Schema

### Scenario (1 hr)
- **Design Challenge:** Create a patient records system that meets these constraints:
  - Full HIPAA compliance with detailed audit trails.
  - Row-level security for different staff roles.
  - Efficient querying for time-series patient measurements.
  - Handling versioned records with complete change history.

### Exercise (15 min)
- Use [dbdiagram.io](https://dbdiagram.io) to draft the schema diagram.
- Discuss trade-offs between security, compliance, and performance.

---

## Day 6: ORM Usage Patterns

### Theory (40 min)
- Overview of ORM best practices (using SQLAlchemy, GORM, or another preferred ORM).
- Understand and avoid common pitfalls like N+1 queries.

### Practical (30 min)
- Implement efficient batch operations and complex joins in a sample application.
- Compare lazy vs. eager loading and assess their trade-offs.

### Focus
- Optimizing ORM usage to prevent performance issues in application code.

---

## Day 7: Caching Strategies

### Theory (40 min)
- Introduction to caching in PostgreSQL: query result caching and materialized views.
- Discussion on when to use internal caching versus external cache systems.

### Practical (30 min)
- Set up a materialized view as a caching mechanism on a sample dataset.
- Evaluate performance improvements using repeated queries.

### Tools
- PostgreSQL materialized view documentation

---

## Day 8: Database Migration Strategies

### Theory (35 min)
- Overview of safe schema changes and the importance of backward compatibility.
- Techniques for zero-downtime migrations.

### Practical (40 min)
- Simulate a complex migration using a tool like Flyway or Liquibase.
- Practice implementing a rollback strategy for error recovery.

### Focus
- Handling legacy code and evolving schemas with minimal disruption.

---

## Day 9: High Availability, Backup & Recovery

### Theory (40 min)
- Explore replication options (synchronous vs. asynchronous) and failover strategies.
- Review backup techniques: logical (pg_dump) and physical backups (pg_basebackup), plus point-in-time recovery (PITR).

### Practical (30 min)
- Configure a primary-replica setup in Docker.
- Simulate a failover scenario and perform a backup/restore exercise.

### Focus
- Ensuring data integrity and system availability in production environments.

---

## Day 10: Practice Scenario – Financial Transaction System & Security Deep Dive

### Recap Exercise (15 min)
- Quick quiz covering key concepts from Days 1–9 (normalization, indexing strategies, query optimization, migration approaches).
- Collaborative review of a complex schema that intentionally contains issues related to:
  - Poor indexing choices.
  - Normalization problems.
  - Security vulnerabilities.
  - Performance bottlenecks.
- Identify and discuss solutions to these issues as a warm-up before the main exercise.

### Scenario (45 min)
- **Design Challenge:** Create a financial ledger system that:
  - Guarantees transactional consistency for money transfers.
  - Supports point-in-time recovery for auditing.
  - Handles high throughput (e.g., 1000+ transactions per second during peak loads).
  - Enables efficient reporting for fraud detection.

### Theory – Security Deep Dive (20 min)
- Brief session on security best practices in PostgreSQL:
  - Encryption (at rest and in transit).
  - Role-based access control and user management.
  - Implementing audit trails and row-level security.

### Practical (15 min)
- Demonstrate concurrency controls and secure access mechanisms within your system.
- **Design Exercise:** Use [dbdiagram.io](https://dbdiagram.io) to visualize your financial system's schema, emphasizing security and transactional integrity.
