# Row-Oriented Databases

**Section:** [Databases](../index.md) › [Relational](index.md)

A **row-oriented** (or row-store) database stores all column values for a single row contiguously on disk. This is the traditional relational database storage model, optimised for transactional workloads that read or write entire rows.

---

## Storage Layout

In a row-oriented layout, a table page stores complete tuples sequentially:

```
Page:
┌────────────────────────────────────────────────────────────────┐
│ [id=1, name="Alice", age=30, email="alice@example.com"]        │
│ [id=2, name="Bob",   age=25, email="bob@example.com"  ]        │
│ [id=3, name="Carol", age=35, email="carol@example.com"]        │
└────────────────────────────────────────────────────────────────┘
```

To retrieve a single row by primary key, the database reads **one page** containing all columns of that row.

---

## Strengths

| Characteristic | Row-Store Advantage |
|----------------|---------------------|
| Point lookups | Entire row retrieved in a single page read |
| OLTP (small reads/writes) | Inserts and updates modify a single page |
| Row-level locking | Concurrency is naturally per-row |
| Index lookups | Secondary indexes point to row TIDs efficiently |

---

## Weaknesses

| Characteristic | Row-Store Disadvantage |
|----------------|------------------------|
| Analytical queries | `SELECT avg(age) FROM users` reads all columns even though only `age` is needed |
| Compression | Adjacent rows have mixed data types — poor compression ratio |
| Vectorised execution | Mixed column types prevent SIMD-optimised batch processing |

For analytical workloads, see [Column-Oriented Databases](column_oriented.md).

---

## Examples

| Database | Notes |
|----------|-------|
| PostgreSQL | Heap storage (row-oriented); columnar possible via extensions like Citus columnar | 
| MySQL InnoDB | Row-oriented with clustered primary key B-Tree |
| SQLite | Row-oriented B-Tree storage |
| Oracle | Row-oriented (default); In-Memory Column Store available as add-on |

---

## Cross-References

- [Column-Oriented Databases](column_oriented.md) — contrasting storage model
- [B-Trees](../../concepts/b_trees.md) — index structure used alongside row storage
- [Disk I/O](../../concepts/disk_io.md) — how page layout affects I/O patterns
- [PostgreSQL](postgresql.md) — canonical row-oriented database implementation
- [ACID Compliance](../../concepts/acid_compliance.md) — transactional guarantees for row-store operations

---

## External Resources

- [The Design and Implementation of Modern Column-Oriented Database Systems (Abadi et al.)](https://www.nowpublishers.com/article/Details/DBS-024)
- [PostgreSQL Heap File Layout](https://www.postgresql.org/docs/current/storage-page-layout.html)
