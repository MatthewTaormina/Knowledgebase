# Memory Hierarchy (RAM vs. Disk)

**Section:** [Concepts](index.md)

The **memory hierarchy** is a tiered model of computer storage where each level trades off capacity, speed, and cost. Understanding this hierarchy is essential for reasoning about database performance, caching strategy, and the motivation behind in-memory databases.

---

## The Storage Pyramid

```
         ┌───────────────────┐
         │   CPU Registers   │  ~0.1 ns   |  bytes
         ├───────────────────┤
         │    L1 Cache       │  ~1 ns     |  32–64 KB
         ├───────────────────┤
         │    L2 Cache       │  ~4 ns     |  256 KB–1 MB
         ├───────────────────┤
         │    L3 Cache       │  ~10 ns    |  8–64 MB
         ├───────────────────┤
         │       DRAM        │  ~100 ns   |  GBs
         ├───────────────────┤
         │  NVMe / SSD       │  ~100 µs   |  TBs
         ├───────────────────┤
         │    HDD (Disk)     │  ~10 ms    |  TBs–PBs
         └───────────────────┘
            Speed ↑    Cost/GB ↑    Capacity ↓
```

---

## Latency Comparison

| Storage Level | Typical Latency | Bandwidth | Relative Cost |
|---------------|----------------|-----------|---------------|
| L1 Cache | ~0.5–1 ns | ~1 TB/s | Very high |
| L2 Cache | ~2–5 ns | ~400 GB/s | High |
| L3 Cache | ~10–40 ns | ~200 GB/s | Medium-high |
| DRAM (Main Memory) | ~60–100 ns | ~50 GB/s | Medium |
| NVMe SSD | ~50–200 µs | ~7 GB/s | Low |
| SATA SSD | ~100–500 µs | ~550 MB/s | Very low |
| HDD | ~5–15 ms | ~100–200 MB/s | Very low |

A RAM access is **~100,000× faster** than an HDD seek. This is why keeping "hot" data in memory is the single most impactful database performance optimisation.

---

## Implications for Databases

### Buffer Pool / Shared Buffers

Databases maintain an in-memory **buffer pool** to cache disk pages. The effective working set must fit in this cache for optimal performance. See [Disk I/O](disk_io.md) for details on page caching.

### In-Memory Databases

Systems like Redis and Memcached store all data in DRAM, eliminating disk I/O for reads entirely. This enables sub-millisecond latency but limits capacity to available RAM.

See [Redis](../databases/in_memory/redis.md) for a real-world example.

### NUMA Considerations

Modern servers have **Non-Uniform Memory Access (NUMA)** architectures: multiple CPU sockets each have local memory. Accessing memory on a remote NUMA node adds ~2–3× latency. Database processes should be pinned to a single NUMA node using `numactl` where possible.

---

## RAM vs. Disk Decision Framework

| Factor | Prefer RAM | Prefer Disk |
|--------|-----------|------------|
| Access latency requirement | < 1 ms | > 1 ms acceptable |
| Dataset size | Fits in server RAM | Too large for RAM |
| Durability requirement | Can tolerate data loss on crash | Must persist across restarts |
| Cost budget | Higher budget per GB | Cost-sensitive, large dataset |

---

## Persistent Memory (PMEM)

Emerging **Non-Volatile DIMM (NVDIMM / Intel Optane)** technology offers DRAM-like latency (~300 ns) with disk-like persistence. Databases are beginning to leverage PMEM as a WAL device or even as primary storage, blurring the traditional RAM/disk boundary.

---

## Cross-References

- [Disk I/O](disk_io.md) — detailed I/O mechanics
- [Storage Mediums](storage_mediums.md) — hardware comparison
- [B-Trees](b_trees.md) — designed around disk page access
- [LSM Trees](lsm_trees.md) — write-optimised structure exploiting sequential I/O
- [Redis](../databases/in_memory/redis.md) — exploits DRAM to eliminate disk I/O

---

## External Resources

- [What Every Programmer Should Know About Memory — Ulrich Drepper](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf)
- [Latency Numbers Every Programmer Should Know](https://gist.github.com/jboner/2841832)
- [Intel Optane Persistent Memory overview](https://www.intel.com/content/www/us/en/products/docs/memory-storage/optane-persistent-memory/overview.html)
