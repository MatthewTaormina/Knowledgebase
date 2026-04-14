# Relational Databases — Index

> **Navigation only.** This section covers relational (SQL) databases, including row-oriented and column-oriented storage models and real-world implementations.

---

## Sub-Folders

| Folder | Description |
|--------|-------------|
| *(none — all documents are flat in this directory)* | |

---

## Documents

| Document | Description |
|----------|-------------|
| [Row-Oriented Databases](row_oriented.md) | Storage model where all columns of a row are stored contiguously — optimal for OLTP |
| [Column-Oriented Databases](column_oriented.md) | Storage model where each column is stored separately — optimal for OLAP and analytics |
| [PostgreSQL](postgresql.md) | Open-source ORDBMS with full ACID compliance, MVCC, and a rich extension ecosystem |

---

## Key Theoretical Cross-References

| Concept | Relevance |
|---------|-----------|
| [B-Trees](../../concepts/b_trees.md) | Default index type for relational databases |
| [ACID Compliance](../../concepts/acid_compliance.md) | Transactional guarantees provided by relational systems |
| [Disk I/O](../../concepts/disk_io.md) | Storage engine design principles |
| [Indexing Strategies](../../concepts/indexing_strategies.md) | Index types available in relational databases |
| [Query Planners](../../concepts/query_planners.md) | SQL query optimisation |
| [Replication](../../concepts/replication.md) | High availability for relational databases |
