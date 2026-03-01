---
title: "Pg Plan Analyzer"
date: 2026-03-01T15:47:00+01:00
description: ""
emoji: "📝"
draft: false
---

Anyone who has debugged a slow PostgreSQL query knows the pain of reading raw `EXPLAIN ANALYZE` output. It's a nested wall of text with costs, row counts, and timings buried inside indentation.

<!--more-->

For simple queries it's manageable, but when you're dealing with multi-join queries across millions of rows, it becomes nearly impossible to spot the problem at a glance.

So I built [pg_plan_analyzer](https://github.com/elie-kazan/pg_plan_analyzer) — a Python tool that takes the JSON output of `EXPLAIN (ANALYZE, FORMAT JSON)` and turns it into an interactive, color-coded execution tree you can explore in your browser.

## How It Works

You start by running your query with JSON plan output in PostgreSQL:

```sql
EXPLAIN (ANALYZE, FORMAT JSON)
SELECT *
FROM orders o
JOIN customers c ON c.id = o.customer_id
ORDER BY o.created_at;
```

Save the output to a JSON file, then run:

```bash
python3 analyzer_2.py test_plan.json
```

The script parses the entire plan tree, analyzes every node for performance issues, and generates two things: a text-based diagnostic summary in your terminal, and `plan.html` — an interactive graph you open in your browser.

![Execution plan graph](/images/plan-graph.png)

## The Visual Graph

This is the core of the tool. Instead of reading plan output line by line, you get a tree diagram where each node represents an operation — sequential scans, hash joins, sorts, gathers, and so on. Each node displays the operation type and actual row count.

The color tells you immediately where the problems are. **Green nodes** are healthy — they're running efficiently with no issues detected. **Red nodes** have warnings — something about that operation needs attention. You don't have to read anything to know where to look first.

Hovering over any red node shows you exactly what's wrong: maybe the planner estimated 1,000 rows but 500,000 came back, or a sort operation spilled to disk, or a hash join exceeded memory and went to multiple batches. The warnings are specific and actionable.

## What It Detects

The analyzer checks every node in the plan for five categories of issues:

**Row estimation errors** — PostgreSQL's query planner makes row count predictions to decide which join strategy and scan method to use. When the planner guesses 100 rows but the actual result is 1,000,000, it's almost certainly chosen the wrong plan. The analyzer compares `Plan Rows` against `Actual Rows` at every node and flags significant mismatches. This is often the root cause of slow queries, and it's one of the hardest things to spot in raw text output.

**Disk-based sorts** — When a `Sort` node runs out of memory, PostgreSQL switches from an in-memory quicksort to an external merge sort on disk. This is dramatically slower but shows up as just a single line in the plan output that's easy to miss. The analyzer flags it immediately, which usually means `work_mem` needs to be increased for that session.

**Hash spills** — Similar problem, different operation. When a `Hash Join` or `Hash Aggregate` can't fit its hash table in memory, it splits into multiple batches and writes the overflow to disk. The analyzer detects when `Hash Batches` exceeds 1 and warns you. Again, typically solved by tuning `work_mem`.

**Expensive nodes** — The analyzer calculates what percentage of total execution time each node consumes. If a single node is eating 80% of the runtime, that's where your optimization effort should go. This saves you from optimizing the wrong thing.

**Missing index suggestions** — When the analyzer sees a `Seq Scan` that reads a large number of rows but returns very few (heavy filtering), it suggests that an index on the filter columns could help. It won't always be the right call — sometimes a sequential scan is the correct choice — but it points you in the right direction.

## Why Not Just Use Existing Tools?

There are web-based tools that visualize plans. They're good, but they require pasting your plan into a browser. I wanted something that runs locally from the terminal, works offline, and gives me opinionated warnings rather than just a visualization.

The goal wasn't to build another plan viewer. It was to build something that tells me **what's wrong and what to do about it** — like having a second pair of DBA eyes reviewing every query plan.

## Try It Out

The project is open source under the MIT license. If you work with PostgreSQL and spend time debugging slow queries, give it a shot:

[github.com/elie-kazan/pg_plan_analyzer](https://github.com/elie-kazan/pg_plan_analyzer)

Run it against one of your slow queries and let me know what you think. I'm actively working on improving the detection heuristics and planning to add plan comparison features down the road.


