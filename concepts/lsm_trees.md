# LSM Trees (Log-Structured Merge Trees)

**Section:** [Concepts](index.md)

A **Log-Structured Merge Tree (LSM Tree)** is a write-optimised data structure that buffers writes in memory and flushes them sequentially to disk, avoiding the random I/O cost of in-place updates. LSM Trees trade read performance for dramatically improved write throughput and are the storage engine of choice for write-heavy workloads.

---

## Motivation

Traditional [B-Tree](b_trees.md) indexes update pages in place: an insert may trigger a page split, and an update overwrites an existing tuple. On HDDs, random writes are extremely expensive (~200 IOPS). On SSDs, in-place updates cause write amplification that degrades both performance and NAND Flash lifespan.

LSM Trees convert all writes into **sequential appends**, exploiting the fact that sequential I/O is orders of magnitude faster than random I/O on all storage media. See [Disk I/O](disk_io.md).

---

## Structure

An LSM Tree has two primary components:

### MemTable (In-Memory Buffer)

Incoming writes are applied to a sorted, in-memory buffer — typically a red-black tree or skip list. This provides O(log n) insert and lookup within the buffer.

### SSTables (Sorted String Tables)

When the MemTable reaches a size threshold, it is **flushed** to disk as an immutable, sorted file called an **SSTable**. SSTables are written sequentially — a single disk write per flush, maximising throughput.

Each SSTable entry contains: `[key | value | sequence number | deletion tombstone]`

---

## Compaction

Over time, multiple SSTables accumulate, potentially containing multiple versions of the same key (updates) or tombstones (deletes). **Compaction** merges SSTables, discards old versions, and removes tombstones, producing a smaller set of clean SSTables.

### Levelled Compaction (LevelDB / RocksDB)

SSTables are organised into levels (L0–Ln). L0 accepts flushed MemTables; SSTables are compacted down into deeper levels where key ranges don't overlap. This limits read amplification to O(levels).

### Size-Tiered Compaction (Cassandra)

SSTables of similar size are merged together. Simple but can cause high space amplification (multiple copies of the same key).

---

## Read Path

To read a key:
1. Check MemTable (in memory — O(log n)).
2. Check SSTables from newest to oldest until the key is found.
3. Bloom filters on each SSTable allow O(1) probabilistic "key not present" checks, skipping irrelevant files.

This multi-step process makes point reads slower than B-Trees, but range scans over recent data remain efficient.

---

## Comparison: LSM Tree vs. B-Tree

| Property | B-Tree | LSM Tree |
|----------|--------|----------|
| Write amplification | Moderate | Low |
| Read amplification | Low (height) | Higher (multiple SSTables) |
| Space amplification | Low | Higher (multiple versions) |
| Best workload | Read-heavy OLTP | Write-heavy, time-series, event logs |
| Random write IOPS | Limited by disk | Very high (buffered in RAM) |
| Real-world engines | PostgreSQL, MySQL | RocksDB, Cassandra, LevelDB, ScyllaDB |

---

## Real-World Implementations

| Database | Engine | Compaction Strategy |
|----------|--------|---------------------|
| Apache Cassandra | Custom (based on LSM) | Size-tiered or levelled (configurable) |
| RocksDB (used by MySQL MyRocks, TiKV) | RocksDB | Levelled |
| LevelDB (Google) | LevelDB | Levelled |
| Apache HBase | HFile (LSM-based) | Minor/major compaction |
| ScyllaDB | Scylla | Levelled / Size-tiered |

---

## Cross-References

- [B-Trees](b_trees.md) — contrasting read-optimised index structure
- [Disk I/O](disk_io.md) — why sequential writes are faster
- [Indexing Strategies](indexing_strategies.md) — where LSM fits in the broader indexing landscape
- [CAP Theorem](cap_theorem.md) — Cassandra's AP trade-offs built on LSM
- [MongoDB](../databases/nosql/mongodb.md) — uses WiredTiger (B-Tree based, not LSM, by default)

---

## External Resources

- [O'Neil et al. (1996) — "The Log-Structured Merge-Tree (LSM-tree)"](https://www.cs.umb.edu/~poneil/lsmtree.pdf) — original paper
- [RocksDB documentation](https://rocksdb.org/blog/2021/05/26/structural-data-structures.html)
- [Designing Data-Intensive Applications — Chapter 3, Martin Kleppmann](https://dataintensive.net/)
