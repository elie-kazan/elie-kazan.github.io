---
title: "Demystifying PostgreSQL EXPLAIN ANALYZE"
date: 2026-01-20
description: "A practical guide to reading query plans and finding bottlenecks in your database."
emoji: "🔍"
---

`EXPLAIN ANALYZE` is the most powerful tool in a DBA's toolkit, yet many developers treat its output like hieroglyphics. Let's break it down.

## What EXPLAIN ANALYZE Actually Does

When you run `EXPLAIN ANALYZE` before a query, PostgreSQL does two things: it executes the query for real, and it instruments every step of the execution plan with actual timing and row count data.

This is different from plain `EXPLAIN`, which only *estimates* what will happen. `ANALYZE` gives you the truth.

```sql
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id)
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE o.created_at > '2025-01-01'
GROUP BY u.name;
```

## Reading the Output

The output is a tree, read from the inside out (bottom to top). Each node shows:

- **Node type** — Seq Scan, Index Scan, Hash Join, etc.
- **Cost** — estimated startup and total cost
- **Rows** — estimated vs actual row count
- **Time** — actual time in milliseconds

The biggest performance wins come from spotting mismatches between estimated and actual rows. When PostgreSQL expects 10 rows but gets 100,000, it's probably chosen the wrong plan.

## Common Red Flags

**Sequential scans on large tables** — usually means a missing index. Not always bad (small tables, or when you need most rows), but worth investigating.

**Nested loops with high row counts** — can be catastrophic. If the outer loop returns 10,000 rows and each triggers an index scan, that's 10,000 individual lookups.

**Sort operations spilling to disk** — check `work_mem`. If sorts are exceeding memory, increasing `work_mem` for the session can help.

## My Workflow

When investigating a slow query, I always follow this pattern: run `EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)` to get the full picture including buffer usage, identify the most expensive node, check if the row estimates are accurate, and then consider whether an index, a rewrite, or a config change is the right fix.

Master this tool and you'll solve 80% of performance problems before they become incidents.
