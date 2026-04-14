# Database Clustering

**Section:** [Databases](../index.md) › [Infrastructure](index.md)

**Database clustering** refers to the deployment of multiple database server instances working together to provide higher availability, increased throughput, and/or greater storage capacity than a single server. This document covers self-managed clustering patterns as an alternative to [managed cloud databases](managed_cloud_databases.md).

---

## Clustering Goals

| Goal | Technique |
|------|-----------|
| High Availability | Active-passive failover with automatic promotion |
| Read Scaling | Read replicas serving read traffic |
| Write Scaling | Sharding or multi-primary configurations |
| Geographic Distribution | Multi-region replication |

---

## Cluster Topologies

### Active-Passive (Primary–Replica)

The most common HA configuration. One node accepts all writes; one or more replicas stay in sync and can be promoted on failure.

```
         ┌────────────┐
Writes → │  Primary   │ ─── WAL stream ──→ ┌─────────────┐
         └────────────┘                    │  Replica(s) │
                                           └─────────────┘
                                                 ↑
                                           Read queries
```

See [Replication](../../concepts/replication.md) for synchronous vs. asynchronous replication trade-offs.

**Tools:**
- PostgreSQL: [Patroni](https://patroni.readthedocs.io/) + etcd/Consul for leader election
- MySQL: [MHA](https://github.com/yoshinorim/mha4mysql-manager), [Orchestrator](https://github.com/openark/orchestrator)

### Active-Active (Multi-Primary)

All nodes accept writes. Requires **conflict resolution** when the same row is updated on two nodes simultaneously.

**Approaches:**
- **Last-Write-Wins (LWW):** Timestamp-based — the later write wins (risk of data loss).
- **Application-level conflict resolution:** The application defines merge logic.
- **CRDTs (Conflict-Free Replicated Data Types):** Mathematical data structures that always converge.

**Examples:** PostgreSQL BDR (Bi-Directional Replication), CockroachDB, Cassandra (AP mode).

### NewSQL Clusters

**NewSQL** databases combine the horizontal scalability of NoSQL with ACID guarantees:

| Database | Architecture |
|----------|-------------|
| **CockroachDB** | Distributed SQL; Raft consensus per range; geo-partitioning |
| **TiDB** | MySQL-compatible; TiKV (Raft + LSM) storage; TiFlash columnar |
| **YugabyteDB** | PostgreSQL-compatible; DocDB storage (LSM-based) |
| **Google Spanner** | TrueTime clock; Paxos consensus; external consistency |

See [Sharding](../../concepts/sharding.md) for the horizontal partitioning theory underpinning these systems.

---

## Consensus Algorithms

Distributed clusters require agreement on which node is the primary and what the committed log looks like. Common consensus algorithms:

| Algorithm | Used By |
|-----------|---------|
| **Raft** | etcd, CockroachDB, TiKV, Consul |
| **Paxos** | Google Spanner (Multi-Paxos), Chubby |
| **Viewstamped Replication** | Academic baseline |
| **Zab (ZooKeeper Atomic Broadcast)** | Apache ZooKeeper, HBase |

---

## Proxy and Load Balancing

Clusters typically deploy a proxy layer to route queries:

| Tool | Purpose |
|------|---------|
| **PgBouncer** | PostgreSQL connection pooler |
| **HAProxy** | TCP/HTTP load balancer; routes to primary vs. replicas |
| **ProxySQL** | MySQL-aware proxy with query routing and caching |
| **Vitess** | MySQL sharding middleware (used by YouTube) |

---

## Monitoring Cluster Health

Key metrics to monitor in a database cluster:

| Metric | Tool |
|--------|------|
| Replication lag | `pg_stat_replication` (PostgreSQL), `SHOW SLAVE STATUS` (MySQL) |
| Failover events | Patroni REST API, Orchestrator API |
| Query throughput & latency | Prometheus + pg_exporter, PMM |
| Disk and memory pressure | Node Exporter, CloudWatch |

---

## Cross-References

- [Replication](../../concepts/replication.md) — the foundation of all clustering patterns
- [Sharding](../../concepts/sharding.md) — write scaling via horizontal partitioning
- [CAP Theorem](../../concepts/cap_theorem.md) — consistency trade-offs in distributed clusters
- [ACID Compliance](../../concepts/acid_compliance.md) — 2PC and distributed transactions
- [Managed Cloud Databases](managed_cloud_databases.md) — cloud-managed alternative to self-managed clusters
- [PostgreSQL](../relational/postgresql.md) — Patroni HA and logical/streaming replication

---

## External Resources

- [Patroni — HA template for PostgreSQL](https://patroni.readthedocs.io/)
- [CockroachDB Architecture](https://www.cockroachlabs.com/docs/stable/architecture/overview.html)
- [TiDB Architecture](https://docs.pingcap.com/tidb/stable/tidb-architecture)
- [Vitess — Database clustering system for MySQL](https://vitess.io/docs/overview/)
- [Raft consensus algorithm (visualised)](https://raft.github.io/)
