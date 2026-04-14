# Disk I/O

**Section:** [Concepts](index.md)

**Disk I/O** (Input/Output) refers to the transfer of data between a computer's main memory (RAM) and a persistent storage device. It is the dominant performance bottleneck in almost all database workloads and is the primary constraint that shapes storage engine design decisions.

---

## Why Disk I/O Matters for Databases

CPU and RAM operate at nanosecond timescales; disk operations occur at microsecond-to-millisecond timescales — a difference of 3–6 orders of magnitude. Minimising the number and cost of disk I/O operations is therefore the central concern of database storage engine design.

See [Memory Hierarchy](memory_hierarchy.md) for a comparison of access latencies across the full storage hierarchy.

---

## Key Concepts

### Pages and Blocks

Operating systems and storage devices do not transfer individual bytes — they read and write fixed-size **blocks**:

| Layer | Unit | Typical Size |
|-------|------|-------------|
| Hardware (HDD/SSD) | Sector | 512 B – 4 KB |
| OS / filesystem | Block | 4 KB |
| Database | Page | 4 KB – 16 KB (PostgreSQL default: 8 KB) |

A database always reads and writes at least one full page, even if only a single byte changed. This property motivates page-level caching (buffer pools) and structures like [B-Trees](b_trees.md) that pack many keys per page.

### Sequential vs. Random I/O

| Access Pattern | HDD Latency | SSD Latency | Notes |
|----------------|-------------|-------------|-------|
| Sequential read | ~100 MB/s | ~500 MB/s–7 GB/s (NVMe) | Disk head moves in one direction |
| Random read (4 KB) | ~1 ms (100 IOPS) | ~0.1 ms (10 000+ IOPS) | HDD seek + rotational latency dominate |

Databases aggressively try to convert random I/O into sequential I/O through:
- **Heap-ordered writes** (e.g., PostgreSQL's sequential scan)
- **Write-ahead logging (WAL)** — all writes are sequential appends
- **Bulk-loading / sorted inserts** — pre-sorting data before writing

### IOPS vs. Throughput

- **IOPS (I/O Operations Per Second):** Measures the number of discrete read/write operations. Critical for OLTP workloads with many small random accesses.
- **Throughput (MB/s):** Measures the volume of data transferred. Critical for OLAP/analytical workloads with large sequential scans.

---

## Buffer Pool / Page Cache

Databases maintain an in-memory **buffer pool** (PostgreSQL: `shared_buffers`) to cache hot pages. On a read:

1. Check the buffer pool — if the page is present (**cache hit**), return it immediately.
2. If not (**cache miss**), fetch from disk, evict a page using a replacement policy (LRU or Clock), and store the new page.

The ratio of cache hits to total requests is the **buffer hit ratio**. A ratio below 95% often signals that `shared_buffers` is undersized.

---

## Write Patterns

### Synchronous Writes

The database waits for the OS/hardware to confirm the write has reached durable storage before acknowledging the transaction. Guarantees [ACID Durability](acid_compliance.md) but adds latency.

### Asynchronous / Buffered Writes

Data is written to an OS page cache and flushed to disk later. Higher throughput, but a crash before the flush loses data.

### fsync and O_DIRECT

- **`fsync`**: Forces OS to flush all dirty pages for a file to disk. Used by PostgreSQL after each WAL segment.
- **`O_DIRECT`**: Bypasses the OS page cache; the database manages its own buffer pool. Used by some engines (e.g., MySQL InnoDB with `innodb_flush_method=O_DIRECT`) to avoid double-buffering.

---

## I/O Scheduling

The Linux kernel I/O scheduler reorders and merges requests to improve throughput:

| Scheduler | Best For |
|-----------|---------|
| `mq-deadline` | General purpose; good for HDDs |
| `none` / `noop` | SSDs and NVMe (no reordering needed) |
| `bfq` | Desktop/interactive workloads |

For SSD/NVMe-backed database servers, `none` is typically recommended.

---

## Cross-References

- [Memory Hierarchy](memory_hierarchy.md) — full latency comparison
- [Storage Mediums](storage_mediums.md) — HDD vs. SSD vs. NVMe hardware details
- [B-Trees](b_trees.md) — index structure designed to minimise page reads
- [LSM Trees](lsm_trees.md) — sequential-write-optimised alternative
- [ACID Compliance](acid_compliance.md) — Durability and its I/O implications

---

## External Resources

- [Database Internals — Alex Petrov (O'Reilly, 2019)](https://www.databass.dev/)
- [Designing Data-Intensive Applications — Martin Kleppmann (O'Reilly, 2017)](https://dataintensive.net/)
- [Linux I/O scheduler documentation](https://www.kernel.org/doc/html/latest/block/index.html)
