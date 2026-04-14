# CAP Theorem

**Section:** [Concepts](index.md)

The **CAP Theorem**, formally proven by Eric Brewer (2000) and later by Gilbert & Lynch (2002), states that a distributed data store can provide at most **two of the following three guarantees** simultaneously:

- **C**onsistency
- **A**vailability
- **P**artition Tolerance

---

## The Three Properties

### Consistency (C)

Every read receives the most recent write or an error. All nodes in the distributed system see the same data at the same time. This is **linearisability** in the formal sense.

### Availability (A)

Every request receives a (non-error) response, even if it may not reflect the most recent write. The system remains operational even when some nodes are down.

### Partition Tolerance (P)

The system continues operating despite arbitrary message loss or delays between nodes (a network partition).

---

## Why You Can Only Choose Two

In a distributed system, **network partitions are unavoidable** — networks fail, packets are dropped, and nodes become temporarily unreachable. Therefore, partition tolerance is not optional in practice.

The real trade-off is: **when a partition occurs, do you sacrifice Consistency or Availability?**

```
          C
         / \
        /   \
       /     \
      CA     CP
       \     /
        \   /
         \ /
          P ─────── AP
```

| Trade-off | Behaviour During Partition | Examples |
|-----------|--------------------------|---------|
| **CP** (Consistent + Partition Tolerant) | Returns an error or times out rather than return stale data | HBase, Zookeeper, etcd, MongoDB (with `majority` writes) |
| **AP** (Available + Partition Tolerant) | Returns potentially stale data rather than error | Cassandra, CouchDB, DynamoDB (eventual consistency) |
| **CA** (Consistent + Available) | Not partition tolerant — only viable in single-node or trusted LAN deployments | Traditional RDBMS on a single node (PostgreSQL, MySQL) |

---

## Beyond CAP: PACELC

The CAP Theorem only describes behaviour during partitions. The **PACELC** model extends it:

- **During a Partition (P):** choose between Availability (A) and Consistency (C).
- **Else (E) — normal operation:** choose between Latency (L) and Consistency (C).

| System | Partition | Else |
|--------|-----------|------|
| DynamoDB | AP | EL (low latency, eventual) |
| Cassandra | AP | EL |
| MongoDB (majority) | CP | EC |
| PostgreSQL | CA | EC |

---

## Consistency Models (Beyond CAP)

CAP uses a strict definition of consistency. In practice, systems offer a spectrum:

| Model | Description |
|-------|-------------|
| **Linearisability** | Strongest — reads always reflect the latest write globally |
| **Sequential Consistency** | All nodes see operations in the same order, but not necessarily real-time |
| **Causal Consistency** | Operations causally related are seen in order; concurrent ops may differ |
| **Eventual Consistency** | All nodes will converge to the same value given no further updates |
| **Read-Your-Writes** | A client always reads its own most recent writes |

---

## Cross-References

- [ACID Compliance](acid_compliance.md) — consistency guarantees within a single-node transaction
- [Sharding](sharding.md) — partitioning introduces partition scenarios
- [Replication](replication.md) — replication lag drives the A vs. C trade-off
- [MongoDB](../databases/nosql/mongodb.md) — configurable CP/AP behaviour
- [PostgreSQL](../databases/relational/postgresql.md) — strong consistency single-node, CA positioning

---

## External Resources

- [Brewer (2000) — "Towards Robust Distributed Systems" (original CAP conjecture)](https://people.eecs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf)
- [Gilbert & Lynch (2002) — Formal proof of CAP Theorem](https://dl.acm.org/doi/10.1145/564585.564601)
- [Kleppmann (2015) — "A Critique of the CAP Theorem"](https://arxiv.org/abs/1509.05393)
- [PACELC Paper — Abadi (2012)](https://cs.yale.edu/homes/abadi/papers/abadi-pacelc.pdf)
