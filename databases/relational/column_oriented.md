# Column-Oriented Databases

**Section:** [Databases](../index.md) › [Relational](index.md)

A **column-oriented** (or columnar) database stores all values for a single column contiguously on disk, rather than storing all columns of a row together. This layout dramatically accelerates analytical queries that aggregate a subset of columns across many rows.

---

## Storage Layout

In a columnar layout, each column is stored as a separate data file or segment:

```
Column "age":   [30, 25, 35, 28, 42, ...]
Column "name":  ["Alice", "Bob", "Carol", "Dave", "Eve", ...]
Column "email": ["alice@…", "bob@…", "carol@…", ...]
```

A query like `SELECT avg(age) FROM users` reads only the `age` column, skipping all other columns entirely.

---

## Strengths

| Characteristic | Column-Store Advantage |
|----------------|------------------------|
| Analytical aggregations | Read only the relevant columns — I/O proportional to columns queried |
| Compression | Homogeneous data type per column — run-length encoding, dictionary encoding, delta encoding |
| Vectorised execution | SIMD CPU instructions can operate on arrays of the same type |
| Late materialisation | Columns are kept separate until the last moment, deferring join cost |

---

## Compression Techniques

| Technique | Best For | Compression Ratio |
|-----------|---------|-------------------|
| Run-Length Encoding (RLE) | Low-cardinality sorted columns | Very high |
| Dictionary Encoding | String columns with repeated values | High |
| Delta / Frame-of-Reference | Numeric/timestamp columns | High |
| Bit-packing | Integer columns with small range | Medium |

Compression reduces both storage cost and I/O volume — fewer pages need to be read.

---

## Weaknesses

| Characteristic | Column-Store Disadvantage |
|----------------|--------------------------|
| Row inserts | A single row insert must update N separate column files |
| Point lookups | Reconstructing a full row requires reading from N column files |
| Updates | In-place updates are expensive — most columnar stores are append-only with compaction |

---

## Examples

| Database | Notes |
|----------|-------|
| Amazon Redshift | Cloud-native columnar data warehouse |
| Google BigQuery | Serverless columnar OLAP |
| Apache Parquet | Columnar file format (not a DBMS) used by Spark, Athena, DuckDB |
| DuckDB | Embedded columnar OLAP database |
| ClickHouse | High-performance columnar OLAP for real-time analytics |
| Snowflake | Cloud data warehouse with columnar storage and separation of compute/storage |

---

## Hybrid (HTAP) Databases

**HTAP (Hybrid Transactional/Analytical Processing)** systems maintain both row and columnar representations:

- **TiDB:** Row store (TiKV) + columnar store (TiFlash) in one system
- **PostgreSQL + Citus Columnar:** Row heap for OLTP; columnar access method for OLAP
- **Oracle In-Memory:** Traditional row store + in-memory column store

---

## Cross-References

- [Row-Oriented Databases](row_oriented.md) — contrasting OLTP-optimised model
- [Disk I/O](../../concepts/disk_io.md) — columnar compression reduces I/O volume
- [Memory Hierarchy](../../concepts/memory_hierarchy.md) — compressed columnar data fits better in CPU cache
- [Indexing Strategies](../../concepts/indexing_strategies.md) — bitmap indexes complement columnar storage

---

## External Resources

- [C-Store: A Column-Oriented DBMS (Stonebraker et al., 2005)](https://vldb.org/archives/website/2005/program/paper/thu/p553-stonebraker.pdf)
- [DuckDB — An Embeddable Analytical Database](https://duckdb.org/why_duckdb)
- [Apache Parquet format specification](https://parquet.apache.org/docs/)
