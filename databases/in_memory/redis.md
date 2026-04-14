# Redis

**Section:** [Databases](../index.md) › [In-Memory](index.md)

**Redis** (Remote Dictionary Server) is an open-source, in-memory data structure store used as a database, cache, message broker, and streaming engine. By keeping all data in RAM, Redis delivers sub-millisecond read and write latencies orders of magnitude lower than disk-based databases.

---

## Theoretical Foundations

- [Memory Hierarchy](../../concepts/memory_hierarchy.md) — Redis exploits DRAM to eliminate disk I/O entirely for reads
- [Disk I/O](../../concepts/disk_io.md) — persistence modes (RDB/AOF) reintroduce controlled, sequential disk writes
- [Replication](../../concepts/replication.md) — Redis replication and Sentinel HA patterns
- [CAP Theorem](../../concepts/cap_theorem.md) — Redis Cluster is AP (available + partition tolerant); standalone Redis is CA

---

## Key Characteristics

| Property | Value |
|----------|-------|
| License | BSD 3-Clause (Redis < 7.4); RSALv2 / SSPLv1 (Redis ≥ 7.4) |
| Data model | Key-value with rich data structures |
| Persistence | Optional (RDB snapshots, AOF log, or both) |
| Replication | Async primary–replica |
| Clustering | Redis Cluster (hash-slot sharding) |
| Latency | < 1 ms (typical) |
| Throughput | 100K–1M+ operations/sec (single node) |

---

## Data Structures

Redis is more than a simple key-value store — it natively supports:

| Type | Description | Example Use Case |
|------|-------------|-----------------|
| **String** | Binary-safe byte sequence (max 512 MB) | Counters, session tokens, HTML fragments |
| **List** | Doubly-linked list of strings | Message queues, activity feeds |
| **Set** | Unordered collection of unique strings | Tagging, unique visitor counts |
| **Sorted Set (ZSet)** | Set with a floating-point score per member | Leaderboards, time-range queries |
| **Hash** | Field-value map | User profiles, session objects |
| **Bitmap** | Bit array operations on strings | Feature flags, daily active user bitmaps |
| **HyperLogLog** | Probabilistic cardinality estimation | Unique page views (±0.81% error) |
| **Stream** | Append-only log with consumer groups | Event sourcing, IoT data ingestion |
| **Geo** | Geospatial index (sorted set with lat/lng encoding) | Proximity searches |

---

## Memory and the Storage Hierarchy

Redis stores the entire active dataset in DRAM. See [Memory Hierarchy](../../concepts/memory_hierarchy.md):

- A typical Redis read completes in **< 100 µs** — limited only by network RTT, not disk.
- Comparison: PostgreSQL heap page read from NVMe ≈ 100 µs; from HDD ≈ 10 ms.
- Redis is ideal when dataset fits in RAM (typical deployments: 1 GB–256 GB).

When memory is exhausted, Redis can evict keys using configurable policies (`allkeys-lru`, `volatile-lfu`, etc.).

---

## Persistence Modes

Redis offers three persistence options:

| Mode | Mechanism | Durability | Performance Impact |
|------|-----------|------------|--------------------|
| **No persistence** | All data in RAM only | None (data lost on restart) | Maximum throughput |
| **RDB (Snapshot)** | Fork + write full dataset to disk at intervals | Moderate (up to `save` interval of data loss) | Minimal (periodic fork) |
| **AOF (Append-Only File)** | Log every write command; replay on startup | High (`appendfsync always` = zero data loss) | Moderate (`fsync` overhead) |
| **RDB + AOF** | Both simultaneously | Highest | Combined overhead |

See [Disk I/O](../../concepts/disk_io.md) for the mechanics of `fsync` and sequential write patterns used by AOF.

---

## High Availability: Redis Sentinel

**Redis Sentinel** provides monitoring, automatic failover, and configuration management for a primary–replica Redis setup:

1. Sentinel processes monitor the primary via heartbeat.
2. On primary failure, Sentinel runs a quorum-based election.
3. A replica is promoted to primary; clients are redirected via Sentinel discovery.

See [Replication](../../concepts/replication.md) for general replication and failover theory.

---

## Horizontal Scaling: Redis Cluster

**Redis Cluster** shards data across up to 1,000 nodes using **hash slots** (16,384 total):

```
hash_slot = CRC16(key) mod 16384
```

Each master node owns a range of hash slots. Clients are redirected via `MOVED` responses. Each master can have replica nodes for HA.

See [Sharding](../../concepts/sharding.md) for horizontal partitioning concepts.

---

## Common Use Cases

| Use Case | Data Structure | Notes |
|----------|---------------|-------|
| Session cache | String / Hash | TTL-based expiry |
| Rate limiting | String + `INCR` + `EXPIRE` | Sliding or fixed window counters |
| Pub/Sub messaging | Pub/Sub / Streams | At-most-once or at-least-once delivery |
| Leaderboards | Sorted Set | O(log n) rank queries |
| Distributed locks | String + `SET NX EX` (Redlock) | Leader election |
| Job queues | List / Streams | BullMQ, Celery Redis backend |
| Full-page caching | String | Store serialised HTML/JSON with TTL |

---

## Cross-References

| Topic | Document |
|-------|---------|
| Why in-memory is fast | [Memory Hierarchy](../../concepts/memory_hierarchy.md) |
| Persistence I/O mechanics | [Disk I/O](../../concepts/disk_io.md) |
| High availability failover | [Replication](../../concepts/replication.md) |
| Cluster sharding | [Sharding](../../concepts/sharding.md) |
| Redis as a managed service | [Managed Cloud Databases](../infrastructure/managed_cloud_databases.md) |
| CAP positioning | [CAP Theorem](../../concepts/cap_theorem.md) |

---

## External Resources

- [Redis Official Documentation](https://redis.io/docs/)
- [Redis in Action — Josiah Carlson (O'Reilly)](https://www.manning.com/books/redis-in-action)
- [Redis Cluster specification](https://redis.io/docs/reference/cluster-spec/)
- [Redlock — Distributed Locking with Redis](https://redis.io/docs/manual/patterns/distributed-locks/)
