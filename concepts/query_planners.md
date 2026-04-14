# Query Planners

**Section:** [Concepts](index.md)

A **query planner** (or query optimiser) is the component of a database engine responsible for determining the most efficient execution strategy for a given SQL statement. It sits between the SQL parser and the execution engine, translating a declarative query into an imperative execution plan.

---

## Query Processing Pipeline

```
SQL Text
   │
   ▼
┌──────────┐    ┌────────────┐    ┌──────────────┐    ┌─────────────┐
│  Parser  │ → │  Rewriter  │ → │   Planner    │ → │  Executor   │
└──────────┘    └────────────┘    └──────────────┘    └─────────────┘
Parse tree       Canonical form    Execution plan        Results
```

1. **Parser:** Validates SQL syntax and produces a parse tree.
2. **Rewriter:** Expands views, applies row-level security policies, and normalises expressions.
3. **Planner:** Enumerates possible execution strategies, estimates their cost, and selects the cheapest.
4. **Executor:** Executes the chosen plan and returns result rows.

---

## Cost-Based Optimisation

Modern query planners use **cost-based optimisation (CBO)**. The planner assigns a numerical cost estimate to each candidate plan based on:

- **Table statistics:** Row counts, column cardinality, histogram data (collected by `ANALYZE` in PostgreSQL).
- **I/O cost model:** `seq_page_cost` (sequential page read) and `random_page_cost` (random page read).
- **CPU cost model:** `cpu_tuple_cost`, `cpu_index_tuple_cost`, `cpu_operator_cost`.

The plan with the lowest total estimated cost is selected.

---

## Key Plan Operations

| Operation | Description |
|-----------|-------------|
| **Sequential Scan** | Read every page of the table from start to finish |
| **Index Scan** | Traverse a [B-Tree](b_trees.md) index, then fetch corresponding heap tuples |
| **Index-Only Scan** | Retrieve all needed data directly from a covering index (no heap access) |
| **Bitmap Heap Scan** | Collect all matching TIDs from an index into a bitmap, then fetch heap pages in order — converts random I/O to sequential |
| **Nested Loop Join** | For each row in the outer relation, probe the inner relation — good for small inner tables with an index |
| **Hash Join** | Build a hash table from the smaller relation, probe with rows from the larger — good for large unsorted joins |
| **Merge Join** | Merge two pre-sorted relations — efficient when both inputs are already sorted |
| **Aggregate / Sort** | In-memory or on-disk sort and group-by operations |

---

## EXPLAIN and EXPLAIN ANALYZE

To inspect the planner's chosen strategy (PostgreSQL example):

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders o
JOIN customers c ON c.id = o.customer_id
WHERE o.created_at > now() - interval '30 days';
```

Key output fields:
- **cost=X..Y**: Estimated startup cost .. total cost.
- **rows=N**: Estimated number of rows returned.
- **actual time=A..B**: Real elapsed time (with ANALYZE).
- **Buffers: shared hit=X read=Y**: Pages served from cache vs. disk.

---

## Statistics and Cardinality Estimation

The planner's accuracy depends on up-to-date statistics:

- PostgreSQL: `ANALYZE table;` — collects per-column histograms, most-common values, and null fractions.
- MySQL: `ANALYZE TABLE table;`
- The `statistics target` (default 100 in PostgreSQL) controls histogram bucket count — higher values improve estimates for skewed data at the cost of more planning time.

Stale statistics are the most common cause of bad query plans.

---

## Adaptive Query Execution

Modern systems (Apache Spark, CockroachDB, DuckDB) use **adaptive query execution**: they start with a plan, observe actual runtime statistics at key points, and re-optimise mid-execution if estimates were wrong.

---

## Cross-References

- [B-Trees](b_trees.md) — primary access method used by Index Scans
- [Indexing Strategies](indexing_strategies.md) — index types the planner can leverage
- [Disk I/O](disk_io.md) — I/O cost model parameters
- [PostgreSQL](../databases/relational/postgresql.md) — EXPLAIN usage and planner configuration

---

## External Resources

- [PostgreSQL — Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
- [CMU 15-445 — Query Planning & Optimization lectures](https://15445.courses.cs.cmu.edu/)
- [Explaining the Postgres Query Optimizer — Bruce Momjian](https://momjian.us/main/writings/pgsql/optimizer.pdf)
