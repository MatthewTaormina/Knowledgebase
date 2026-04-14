# In-Memory Databases — Index

> **Navigation only.** This section covers databases that store their primary dataset in RAM, enabling sub-millisecond access latencies.

---

## Sub-Folders

| Folder | Description |
|--------|-------------|
| *(none — all documents are flat in this directory)* | |

---

## Documents

| Document | Description |
|----------|-------------|
| [Redis](redis.md) | In-memory key-value store with persistence options, data structures, and pub/sub |

---

## Key Theoretical Cross-References

| Concept | Relevance |
|---------|-----------|
| [Memory Hierarchy](../../concepts/memory_hierarchy.md) | In-memory databases exploit DRAM to eliminate disk I/O latency |
| [Disk I/O](../../concepts/disk_io.md) | Persistence modes (RDB/AOF) reintroduce controlled disk writes |
| [Replication](../../concepts/replication.md) | Redis Sentinel and Cluster replication patterns |
| [CAP Theorem](../../concepts/cap_theorem.md) | Redis Cluster is AP by default |
