# Concepts — Index

> **Navigation only.** This section contains reusable, standalone theoretical documents covering hardware/storage fundamentals, software optimisation algorithms, and distributed systems principles. These documents are cross-referenced by implementation-specific documents elsewhere in the knowledgebase.

---

## Sub-Folders

| Folder | Description |
|--------|-------------|
| *(none — all documents are flat in this directory)* | |

---

## Documents

### Hardware & Storage

| Document | Description |
|----------|-------------|
| [Disk I/O](disk_io.md) | How data is read from and written to disk; latency, throughput, and optimisation techniques |
| [Memory Hierarchy (RAM vs. Disk)](memory_hierarchy.md) | CPU caches, DRAM, SSDs, and HDDs — the storage pyramid and its performance implications |
| [Storage Mediums](storage_mediums.md) | Comparison of HDD, SSD (NAND Flash), NVMe, and emerging persistent-memory technologies |

### Software Optimisation & Algorithms

| Document | Description |
|----------|-------------|
| [B-Trees](b_trees.md) | Self-balancing tree structure; the default index type in most relational databases |
| [LSM Trees](lsm_trees.md) | Log-Structured Merge Trees; write-optimised structure used by RocksDB, Cassandra, and LevelDB |
| [Indexing Strategies](indexing_strategies.md) | Overview of index types — B-Tree, Hash, Bitmap, GIN, GiST, covering vs. partial indexes |
| [Query Planners](query_planners.md) | How databases parse, optimise, and execute SQL queries using cost-based planning |

### Distributed Systems

| Document | Description |
|----------|-------------|
| [CAP Theorem](cap_theorem.md) | Consistency, Availability, and Partition Tolerance trade-offs in distributed databases |
| [ACID Compliance](acid_compliance.md) | Atomicity, Consistency, Isolation, and Durability — the guarantees of transactional databases |
| [Sharding](sharding.md) | Horizontal partitioning strategies for distributing data across multiple nodes |
| [Replication](replication.md) | Synchronous and asynchronous replication patterns for fault tolerance and read scaling |
