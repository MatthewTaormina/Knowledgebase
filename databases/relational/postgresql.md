# PostgreSQL

**Section:** [Databases](../index.md) › [Relational](index.md)

**PostgreSQL** (pronounced *post-gres-Q-L*, often shortened to *Postgres*) is a free and open-source, ACID-compliant object-relational database management system (ORDBMS). Originally developed at the University of California, Berkeley as POSTGRES (1986), it has evolved into one of the most feature-rich and standards-compliant SQL databases available.

---

## Theoretical Foundations

PostgreSQL's architecture is grounded in several core theoretical concepts. Before reading this document, it is recommended to review:

- [B-Trees](../../concepts/b_trees.md) — PostgreSQL's default and most widely used index structure
- [ACID Compliance](../../concepts/acid_compliance.md) — the transactional guarantees PostgreSQL provides and how they are implemented
- [Disk I/O](../../concepts/disk_io.md) — the hardware model PostgreSQL's storage engine is optimised around
- [Indexing Strategies](../../concepts/indexing_strategies.md) — the full range of index types PostgreSQL supports beyond B-Trees
- [Query Planners](../../concepts/query_planners.md) — how PostgreSQL's cost-based planner selects execution strategies

---

## Key Characteristics

| Property | Value |
|----------|-------|
| License | PostgreSQL License (permissive open-source) |
| Data model | Relational (row-oriented heap storage) |
| Query language | SQL (SQL:2023 compliant subset) |
| ACID | Full support via MVCC + WAL |
| Default index | B+ Tree |
| Replication | Streaming replication (physical), logical replication |
| Concurrency | Multi-Version Concurrency Control (MVCC) |
| Extensions | PostGIS, pgvector, TimescaleDB, Citus, and hundreds more |
| Latest stable | 16.x (as of 2024) |

---

## Storage Architecture

### Heap Storage (Row-Oriented)

PostgreSQL stores table data in **heap files** — a row-oriented format where each page (default 8 KB) contains a contiguous array of tuples (rows). Each tuple is self-contained: it stores all column values for a single row.

```
Page Layout (8 KB default):
┌────────────────────────────────────────────────────┐
│ Page Header (24 bytes)                             │
│ Item Pointers (array of (offset, length) pairs)    │
│                     ↕ free space ↕                 │
│ Tuple N … Tuple 2 … Tuple 1 (grow from bottom)    │
│ Special Space (index-specific metadata)            │
└────────────────────────────────────────────────────┘
```

This contrasts with column-oriented storage (see [Column-Oriented Databases](column_oriented.md)) where columns are stored separately, making PostgreSQL better suited for OLTP workloads that read entire rows.

See [Disk I/O](../../concepts/disk_io.md) for how page size interacts with OS and hardware block sizes.

### Tablespaces

PostgreSQL supports **tablespaces** — mapping logical storage to physical directories/volumes. This allows:
- Placing frequently accessed data on fast NVMe storage
- Archiving cold data to slower, cheaper HDDs
- Separating indexes from table data across devices

See [Storage Mediums](../../concepts/storage_mediums.md) for a hardware comparison relevant to tablespace design.

---

## ACID Implementation

PostgreSQL achieves full [ACID compliance](../../concepts/acid_compliance.md) through two complementary mechanisms:

### 1. Write-Ahead Logging (WAL)

Before any change is applied to a data page, PostgreSQL writes a record to the **WAL** (also called the transaction log or `pg_wal` directory). This provides:

- **Durability (D):** On crash, WAL records are replayed to recover committed transactions.
- **Atomicity (A):** An incomplete transaction's WAL records are discarded on recovery.

WAL records contain sufficient information to reconstruct the before and after state of each modified page.

### 2. Multi-Version Concurrency Control (MVCC)

PostgreSQL never overwrites existing row data in place. Instead, each `UPDATE` creates a **new tuple version** (with a new transaction ID in `xmin`) while marking the old version as deleted (setting `xmax`). A `DELETE` simply sets `xmax`.

This provides:

- **Isolation (I):** Readers never block writers; each transaction sees a consistent snapshot of the database as of its start time.
- **Consistency (C):** Constraints and triggers enforce data integrity; MVCC ensures concurrent transactions cannot see each other's uncommitted changes.

**MVCC trade-off:** Old tuple versions accumulate and must be periodically reclaimed by `VACUUM` to prevent table bloat and transaction ID wraparound.

### Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|---------------------|--------------|
| Read Committed (default) | No | Possible | Possible |
| Repeatable Read | No | No | No (PostgreSQL-specific) |
| Serializable | No | No | No |

PostgreSQL implements Serializable via **Serializable Snapshot Isolation (SSI)** — a predicate-locking scheme that detects serialisation anomalies without blocking.

---

## Indexing with B-Trees

PostgreSQL's default and most versatile index type is the [B+ Tree](../../concepts/b_trees.md). Key properties in PostgreSQL's implementation:

- **Index entries** store the indexed column value(s) and a **Tuple Identifier (TID)** pointing to the heap page and offset of the corresponding row.
- Supports `=`, `<`, `<=`, `>`, `>=`, `BETWEEN`, `IN`, `LIKE 'prefix%'`, and `IS NULL` / `IS NOT NULL` operators.
- Multi-column (composite) indexes maintain keys sorted lexicographically by the first column, then subsequent columns.

### Index Types Available

See [Indexing Strategies](../../concepts/indexing_strategies.md) for full detail. Summary for PostgreSQL:

| Index Type | AM Name | Best For |
|------------|---------|---------|
| B-Tree | `btree` | Equality, range, ordering — the universal default |
| Hash | `hash` | Equality only; slightly faster for `=` point lookups |
| GIN | `gin` | Full-text search, JSONB containment, array operators |
| GiST | `gist` | Geometric/spatial data, full-text, custom types |
| SP-GiST | `spgist` | Space-partitioned data (quad-trees, k-d trees) |
| BRIN | `brin` | Very large tables with naturally ordered data (e.g., time-series) — tiny index size |

### Partial Indexes

PostgreSQL supports **partial indexes** — an index built over a subset of rows defined by a `WHERE` clause:

```sql
CREATE INDEX idx_orders_pending ON orders (created_at)
WHERE status = 'pending';
```

This is far smaller than a full index and only consulted when the query's `WHERE` clause matches the index predicate.

### Covering Indexes (Index-Only Scans)

By adding `INCLUDE` columns, a B-Tree index can serve a query entirely without accessing the heap:

```sql
CREATE INDEX idx_users_email ON users (email) INCLUDE (name, created_at);
```

---

## Query Planning

PostgreSQL's **planner/optimiser** is a cost-based system. For each query it:

1. **Parses** SQL into a parse tree.
2. **Rewrites** using rule system (views, security policies).
3. **Plans** — enumerates execution strategies and estimates their cost using statistics from `pg_statistic` (populated by `ANALYZE`).
4. **Executes** using the lowest-cost plan.

See [Query Planners](../../concepts/query_planners.md) for the general theory. To inspect PostgreSQL's chosen plan:

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) SELECT * FROM orders WHERE customer_id = 42;
```

Key planner concepts:
- **`seq_page_cost` / `random_page_cost`**: Relative cost of sequential vs. random disk reads. Reducing `random_page_cost` (e.g., to 1.1 for SSDs) encourages more index usage.
- **Statistics target**: `ALTER TABLE … ALTER COLUMN … SET STATISTICS n` increases histogram resolution for better cardinality estimates.
- **Parallel query**: PostgreSQL 9.6+ can parallelise sequential scans, hash joins, and aggregations across multiple CPU cores.

---

## Replication

PostgreSQL supports two replication modes. See [Replication](../../concepts/replication.md) for the general theory:

| Mode | Description | Use Case |
|------|-------------|---------|
| **Streaming Replication** (physical) | WAL bytes streamed to standbys; byte-for-byte copy | High availability, read replicas |
| **Logical Replication** | Row-level change stream (insert/update/delete) | Cross-version replication, selective table sync |

Synchronous vs. asynchronous replication is configurable via `synchronous_commit`:
- `on` (default): wait for WAL to be flushed on standby — guarantees no data loss.
- `off`: return immediately — up to 1 WAL segment of data loss on standby failure.

---

## Concurrency Tuning

| Parameter | Description | Typical Value |
|-----------|-------------|---------------|
| `max_connections` | Maximum concurrent client connections | 100–500 (use PgBouncer for pooling) |
| `shared_buffers` | PostgreSQL's in-memory page cache | 25% of RAM |
| `work_mem` | Memory per sort/hash operation | 4 MB–256 MB |
| `wal_buffers` | In-memory WAL buffer | 64 MB |
| `effective_cache_size` | Planner hint for total OS + PG cache | 75% of RAM |

---

## Extensions Ecosystem

PostgreSQL's extension mechanism allows entire storage engines and data types to be added:

| Extension | Purpose |
|-----------|---------|
| **PostGIS** | Geospatial data types and functions |
| **pgvector** | Vector similarity search (embeddings) |
| **TimescaleDB** | Time-series optimisations on PostgreSQL |
| **Citus** | Transparent sharding — see [Sharding](../../concepts/sharding.md) |
| **pg_partman** | Declarative table partitioning management |
| **pglogical** | Advanced logical replication |

---

## Cross-References

| Topic | Document |
|-------|---------|
| Index internals | [B-Trees](../../concepts/b_trees.md) |
| Transactional guarantees | [ACID Compliance](../../concepts/acid_compliance.md) |
| Storage hardware considerations | [Disk I/O](../../concepts/disk_io.md) |
| Memory tuning | [Memory Hierarchy](../../concepts/memory_hierarchy.md) |
| Index type selection | [Indexing Strategies](../../concepts/indexing_strategies.md) |
| Query plan inspection | [Query Planners](../../concepts/query_planners.md) |
| High-availability setup | [Replication](../../concepts/replication.md) |
| Horizontal scaling | [Sharding](../../concepts/sharding.md) |
| Row vs. column storage | [Row-Oriented Databases](row_oriented.md) |
| Cloud-hosted PostgreSQL options | [Managed Cloud Databases](../infrastructure/managed_cloud_databases.md) |

---

## External Resources

- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/current/)
- [The Internals of PostgreSQL (Hironobu Suzuki)](https://www.interdb.jp/pg/)
- [PostgreSQL Wiki — MVCC](https://wiki.postgresql.org/wiki/MVCC)
- [pgBadger — PostgreSQL log analyser](https://pgbadger.darold.net/)
- [pgBouncer — Connection Pooler](https://www.pgbouncer.org/)
- [USE THE INDEX, LUKE — SQL indexing guide](https://use-the-index-luke.com/)
