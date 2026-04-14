# Indexing Strategies

**Section:** [Concepts](index.md)

An **index** is an auxiliary data structure maintained alongside a database table that enables fast data retrieval without scanning every row. Choosing the right index type is one of the highest-impact database performance decisions.

---

## Why Indexes Exist

Without an index, the database must perform a **sequential scan** — reading every page of a table to find matching rows (O(n)). An index provides a pre-sorted or pre-grouped lookup structure that narrows the search to O(log n) or O(1) depending on type.

The cost: indexes consume additional storage and must be updated on every write, adding write overhead.

---

## Index Types

### B-Tree Index

The universal default. Supports equality (`=`), range (`<`, `>`, `BETWEEN`), sorting (`ORDER BY`), and prefix matching (`LIKE 'abc%'`).

See [B-Trees](b_trees.md) for full technical detail.

**Best for:** OLTP workloads, primary keys, foreign keys, range queries.

### Hash Index

Stores a hash of the index key mapped to the row location. Provides O(1) equality lookups but **cannot** support range queries or ordering.

**Best for:** Exact-match lookups where range queries are never needed (rare in practice — B-Trees are usually preferred).

### Bitmap Index

Stores one bit per row per distinct value. A bitmap for `status = 'active'` has a `1` bit for every row where status is active. Multiple bitmaps can be ANDed/ORed efficiently.

**Best for:** Low-cardinality columns in OLAP/data warehousing (e.g., `status`, `country`, `category`).

**Limitations:** Expensive to update — not suitable for OLTP.

### GIN (Generalised Inverted Index)

Stores a mapping from each element of a composite value (array, JSON key, text lexeme) to the rows containing it. Similar to a search engine's inverted index.

**Best for:** Full-text search, JSONB containment queries (`@>`), array operators (`&&`, `@>`).

### GiST (Generalised Search Tree)

A framework for building custom index types using user-defined strategies. Used for geometric/spatial data, range types, and full-text (`tsvector`).

**Best for:** PostGIS spatial queries, proximity searches, custom data types.

### BRIN (Block Range INdex)

Stores the min/max values for a range of adjacent table blocks. Extremely small (typically kilobytes regardless of table size) but only useful when data is physically ordered by the indexed column.

**Best for:** Very large tables where data is naturally ordered (e.g., time-series append-only tables, auto-incrementing IDs).

---

## Index Features

### Composite (Multi-Column) Indexes

An index on multiple columns: `(a, b, c)`. Useful for queries filtering on the leading columns. The column order matters — an index on `(a, b)` can satisfy `WHERE a = ?` but not `WHERE b = ?` alone.

### Partial Indexes

An index with a `WHERE` clause that only indexes a subset of rows:

```sql
CREATE INDEX idx_active_users ON users (email) WHERE active = true;
```

Smaller and faster than a full index for selective queries.

### Covering Indexes

An index that includes all columns needed by a query, eliminating the need to access the heap:

```sql
CREATE INDEX idx_orders ON orders (customer_id) INCLUDE (amount, created_at);
```

Enables **Index-Only Scans** in PostgreSQL.

### Clustered vs. Non-Clustered Indexes

- **Clustered:** The table data is physically ordered according to the index key. MySQL InnoDB's primary key is always clustered.
- **Non-clustered:** The index is a separate structure pointing to the heap; the heap has no particular order.

PostgreSQL does not have traditional clustered indexes but supports `CLUSTER table USING index` as a one-time physical reorder.

---

## Index Selection Guidelines

| Query Pattern | Recommended Index |
|---------------|------------------|
| `WHERE col = ?` | B-Tree or Hash |
| `WHERE col BETWEEN ? AND ?` | B-Tree |
| `ORDER BY col` | B-Tree |
| Full-text search | GIN (tsvector) |
| JSONB containment | GIN |
| Geospatial queries | GiST (PostGIS) |
| Low-cardinality OLAP | Bitmap |
| Append-only time-series | BRIN |

---

## Cross-References

- [B-Trees](b_trees.md) — detailed B-Tree mechanics
- [LSM Trees](lsm_trees.md) — alternative to B-Tree for write-heavy workloads
- [Query Planners](query_planners.md) — how the database chooses which index to use
- [PostgreSQL](../databases/relational/postgresql.md) — supports all index types listed above

---

## External Resources

- [USE THE INDEX, LUKE — comprehensive SQL indexing guide](https://use-the-index-luke.com/)
- [PostgreSQL Index Types documentation](https://www.postgresql.org/docs/current/indexes-types.html)
- [MySQL Index Optimization](https://dev.mysql.com/doc/refman/8.0/en/optimization-indexes.html)
