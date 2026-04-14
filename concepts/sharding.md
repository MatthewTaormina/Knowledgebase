# Sharding

**Section:** [Concepts](index.md)

**Sharding** (also called **horizontal partitioning**) is the technique of distributing data across multiple database nodes, each holding a subset of the total dataset. It is the primary mechanism for scaling databases beyond the capacity of a single machine.

---

## Motivation

A single database server has finite CPU, RAM, disk, and network capacity. When a dataset grows beyond these limits, two strategies exist:

- **Vertical scaling (scale-up):** Upgrade to a larger server. Limited by hardware maximums and expensive.
- **Horizontal scaling (scale-out / sharding):** Add more servers, each holding a portion of the data. Theoretically unlimited.

---

## Sharding Strategies

### Range-Based Sharding

Rows are assigned to shards based on a range of the shard key value.

```
Shard 1: user_id 1 – 1,000,000
Shard 2: user_id 1,000,001 – 2,000,000
Shard 3: user_id 2,000,001 – 3,000,000
```

**Pros:** Simple; range queries on the shard key hit only one shard.  
**Cons:** **Hotspots** if data is not uniformly distributed (e.g., new users always go to the last shard).

### Hash-Based Sharding

The shard key is hashed, and the hash modulo the number of shards determines placement:

```
shard = hash(user_id) % num_shards
```

**Pros:** Even data distribution; eliminates hotspots.  
**Cons:** Range queries must fan out to all shards. Resharding requires rebalancing all data.

### Consistent Hashing

A hash ring is used so that when a shard is added or removed, only a fraction of keys need to be remapped (1/n on average). Used by DynamoDB, Cassandra, and Riak.

### Directory-Based Sharding

A lookup service maps each key to its shard. Most flexible but the directory becomes a bottleneck and single point of failure.

---

## Cross-Shard Queries and Distributed Transactions

Sharding introduces significant complexity:

- **Scatter-gather queries:** A query without the shard key must be broadcast to all shards and results merged.
- **Cross-shard joins:** Must be performed at the application layer or coordinator — expensive.
- **Distributed transactions:** Maintaining [ACID compliance](acid_compliance.md) across shards requires a two-phase commit (2PC) protocol, which adds latency and is a potential single point of failure.

For this reason, schemas are typically designed so that related data co-locates on the same shard (e.g., all orders for a given `customer_id` on the same shard).

---

## Rebalancing

When shards become uneven (new data grows unevenly, or new shards are added), data must be **rebalanced** — moved between shards without downtime. This is a complex operational challenge.

Consistent hashing minimises rebalancing. Systems like Vitess (MySQL sharding) and Citus (PostgreSQL sharding) automate this.

---

## Sharding vs. Partitioning

| Term | Scope | Location |
|------|-------|---------|
| Sharding | Distributes data across **multiple servers** | Different machines |
| Partitioning | Divides a table into segments on **one server** | Same machine, different files |

PostgreSQL supports declarative table partitioning (range, list, hash) on a single node. Citus extends this to sharding across multiple nodes.

---

## Cross-References

- [CAP Theorem](cap_theorem.md) — partition tolerance implications
- [ACID Compliance](acid_compliance.md) — distributed transactions complicate ACID
- [Replication](replication.md) — each shard typically has its own replica set
- [MongoDB](../databases/nosql/mongodb.md) — built-in sharding via mongos router
- [PostgreSQL](../databases/relational/postgresql.md) — Citus extension for PostgreSQL sharding

---

## External Resources

- [Designing Data-Intensive Applications — Chapter 6, Martin Kleppmann](https://dataintensive.net/)
- [Vitess — MySQL sharding at scale](https://vitess.io/)
- [Citus — PostgreSQL sharding extension](https://www.citusdata.com/)
- [Cassandra — Consistent Hashing architecture](https://cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html)
