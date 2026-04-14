# Databases — Index

> **Navigation only.** This section covers database paradigms, storage engines, and real-world implementations. Theoretical underpinnings (B-Trees, ACID, CAP Theorem, etc.) are documented in the canonical [Concepts](../concepts/index.md) directory and cross-referenced from the documents below.

---

## Sub-Folders

| Folder | Description |
|--------|-------------|
| [relational/](relational/index.md) | Row-oriented and column-oriented relational databases (SQL) |
| [nosql/](nosql/index.md) | Non-relational databases: document stores, key-value, wide-column, and graph |
| [in_memory/](in_memory/index.md) | In-memory data stores optimised for low-latency access |
| [infrastructure/](infrastructure/index.md) | Cloud-managed database services and self-managed clustering patterns |

---

## Overview

Databases are organised storage systems that allow structured retrieval, modification, and persistence of data. This domain is divided along two axes:

**By data model:**

| Model | Paradigm | Examples |
|-------|----------|---------|
| Relational | Tables, rows, SQL | PostgreSQL, MySQL, CockroachDB |
| Document | JSON / BSON documents | MongoDB, CouchDB, Firestore |
| Key-Value | Hash-map semantics | Redis, DynamoDB, etcd |
| Wide-Column | Sparse column families | Cassandra, HBase, Bigtable |
| Graph | Nodes and edges | Neo4j, Amazon Neptune |
| Time-Series | Append-optimised temporal data | InfluxDB, TimescaleDB |

**By storage location:**

| Location | Characteristics | Examples |
|----------|-----------------|---------|
| Disk-based | Durable, high capacity | PostgreSQL, MongoDB |
| In-memory | Ultra-low latency, limited by RAM | Redis, Memcached |
| Hybrid | Hot data in RAM, cold on disk | Apache Ignite, VoltDB |

---

## Key Theoretical Cross-References

The following foundational concepts underpin all database paradigms. Consult these documents before diving into specific implementations:

| Concept | Relevance |
|---------|-----------|
| [B-Trees](../concepts/b_trees.md) | Primary index structure in relational and many NoSQL databases |
| [LSM Trees](../concepts/lsm_trees.md) | Write-optimised alternative used by document stores and wide-column engines |
| [ACID Compliance](../concepts/acid_compliance.md) | Transactional guarantees required by relational databases |
| [CAP Theorem](../concepts/cap_theorem.md) | Fundamental trade-off governing distributed database design |
| [Sharding](../concepts/sharding.md) | Horizontal scaling technique used by NoSQL and NewSQL systems |
| [Replication](../concepts/replication.md) | High-availability and read-scaling pattern common to all paradigms |
| [Disk I/O](../concepts/disk_io.md) | Hardware constraint that shapes storage engine design |
| [Memory Hierarchy](../concepts/memory_hierarchy.md) | Performance hierarchy from CPU cache to disk — critical for in-memory databases |
| [Indexing Strategies](../concepts/indexing_strategies.md) | Survey of index types applicable across paradigms |
| [Query Planners](../concepts/query_planners.md) | Cost-based query optimisation common to SQL and some NoSQL engines |
