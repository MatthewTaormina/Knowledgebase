# Replication

**Section:** [Concepts](index.md)

**Replication** is the process of maintaining copies of data on multiple database nodes. It is the foundation of high availability, disaster recovery, and read scaling in database systems.

---

## Goals of Replication

| Goal | Description |
|------|-------------|
| **High Availability** | If the primary fails, a replica can be promoted with minimal downtime |
| **Disaster Recovery** | A geographically separate replica survives datacenter failures |
| **Read Scaling** | Read queries can be distributed across replicas, offloading the primary |
| **Zero-Downtime Maintenance** | Maintenance can be performed on the primary while a replica serves traffic |

---

## Synchronous vs. Asynchronous Replication

### Synchronous

The primary waits for at least one replica to acknowledge the write before confirming to the client.

- **Pro:** Zero data loss on primary failure — replica is always up-to-date.
- **Con:** Write latency increases (network round-trip to replica); a slow/unresponsive replica can block all writes.

### Asynchronous

The primary confirms to the client immediately; replication happens in the background.

- **Pro:** Low write latency; replica failure does not block the primary.
- **Con:** **Replication lag** — if the primary fails before the replica catches up, committed writes may be lost (**replica lag = potential data loss window**).

### Semi-Synchronous (MySQL)

At least one replica acknowledges receipt of the WAL record before the primary commits. A middle ground between full sync and async.

---

## Replication Methods

### Physical (Binary / WAL-based) Replication

The primary streams raw WAL bytes (PostgreSQL) or binary log events (MySQL binlog) to replicas. Replicas replay the changes, maintaining a byte-for-byte copy of the primary.

- **Pros:** Efficient; replicas are always exact copies.
- **Cons:** Replicas must run the same database version and architecture; cannot replicate to a different database engine.

### Logical Replication

The primary streams row-level change events (insert/update/delete per table) rather than raw bytes.

- **Pros:** Replicate to different database versions, different schemas (transformed replication), or different database engines.
- **Cons:** Overhead of decoding WAL into logical events; DDL changes may require manual handling.

---

## Replication Topologies

| Topology | Description | Use Case |
|----------|-------------|---------|
| **Primary–Replica (Single Primary)** | One writable primary; N read-only replicas | Most OLTP systems |
| **Multi-Primary (Active–Active)** | Multiple writable primaries; conflict resolution required | Geo-distributed writes |
| **Cascading Replication** | Replica streams to downstream replicas (reduces primary load) | Large replica fan-out |
| **Ring Replication** | Circular chain of primaries | Rarely used; complex conflict resolution |

---

## Failover

When the primary fails:
1. A replica is **promoted** to primary (automatic with Patroni/Repmgr for PostgreSQL, or MHA for MySQL).
2. Applications reconnect to the new primary (via a virtual IP or load balancer).
3. Other replicas reconfigure to replicate from the new primary.
4. The old primary is repaired and rejoins as a replica.

The **Recovery Point Objective (RPO)** is the maximum data loss acceptable. Synchronous replication achieves RPO = 0.

The **Recovery Time Objective (RTO)** is the maximum downtime acceptable. Automated failover tools achieve RTO of 10–60 seconds.

---

## Replication Lag

Replication lag is the delay between a write on the primary and its appearance on a replica. Causes:
- Network latency
- Replica CPU/disk saturation (replay bottleneck)
- Long-running transactions on the replica (PostgreSQL: replication slot bloat)

Monitoring replication lag (PostgreSQL):
```sql
SELECT
  application_name,
  pg_wal_lsn_diff(pg_current_wal_lsn(), sent_lsn)  AS send_lag_bytes,
  pg_wal_lsn_diff(sent_lsn, replay_lsn)             AS replay_lag_bytes
FROM pg_stat_replication;
```

---

## Cross-References

- [CAP Theorem](cap_theorem.md) — replication lag is at the heart of the C vs. A trade-off
- [ACID Compliance](acid_compliance.md) — synchronous replication extends durability guarantees
- [Sharding](sharding.md) — each shard typically has its own replica set
- [PostgreSQL](../databases/relational/postgresql.md) — streaming and logical replication implementation

---

## External Resources

- [Designing Data-Intensive Applications — Chapter 5, Martin Kleppmann](https://dataintensive.net/)
- [PostgreSQL — High Availability and Replication](https://www.postgresql.org/docs/current/high-availability.html)
- [Patroni — HA template for PostgreSQL](https://patroni.readthedocs.io/)
- [MySQL Replication documentation](https://dev.mysql.com/doc/refman/8.0/en/replication.html)
