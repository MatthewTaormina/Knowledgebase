# MongoDB

**Section:** [Databases](../index.md) › [NoSQL](index.md)

**MongoDB** is an open-source, document-oriented NoSQL database developed by MongoDB, Inc. It stores data as flexible JSON-like BSON documents, provides rich querying, aggregation, full-text search, and built-in horizontal [sharding](../../concepts/sharding.md).

---

## Theoretical Foundations

- [Document Stores](document_stores.md) — MongoDB's data model
- [CAP Theorem](../../concepts/cap_theorem.md) — MongoDB is configurable CP/AP depending on write concern and read preference
- [B-Trees](../../concepts/b_trees.md) — WiredTiger's default index structure
- [Sharding](../../concepts/sharding.md) — MongoDB's native horizontal scaling via mongos
- [Replication](../../concepts/replication.md) — MongoDB Replica Sets

---

## Key Characteristics

| Property | Value |
|----------|-------|
| License | Server Side Public License (SSPL) |
| Data model | Document (BSON / JSON) |
| Query language | MongoDB Query Language (MQL) |
| ACID | Multi-document ACID since v4.0 (via transactions) |
| Default storage engine | WiredTiger (B-Tree based) |
| Replication | Replica Sets (one primary, N secondaries) |
| Sharding | Built-in via mongos router |
| Latest stable | 7.x (as of 2024) |

---

## Data Model

### BSON Documents

MongoDB stores data as **BSON** (Binary JSON) — a superset of JSON that adds additional data types (ObjectId, Date, Binary, Decimal128):

```json
{
  "_id": { "$oid": "65c1a2b3d4e5f6a7b8c9d0e1" },
  "title": "Introduction to MongoDB",
  "tags": ["nosql", "database"],
  "author": { "name": "Alice", "email": "alice@example.com" },
  "published_at": { "$date": "2024-01-15T10:00:00Z" },
  "view_count": 1523
}
```

### Collections and Databases

Documents are grouped into **collections** (analogous to tables) within **databases** (analogous to schemas). No schema declaration is required.

---

## WiredTiger Storage Engine

MongoDB's default storage engine since v3.2. Key features:

- **B-Tree indexes** for default index structure (see [B-Trees](../../concepts/b_trees.md))
- **Document-level concurrency control** — no table-level locking
- **Snappy/zstd compression** for both data and indexes
- **MVCC snapshots** for read isolation
- **Journal (WAL)** for crash recovery and durability

---

## Indexing

MongoDB supports a rich set of index types:

| Index Type | Description |
|------------|-------------|
| Single Field | B-Tree index on one field |
| Compound | B-Tree index on multiple fields |
| Multikey | Indexes array fields — one entry per array element |
| Text | Inverted index for full-text search |
| Geospatial (2d/2dsphere) | GiST-like spatial indexing |
| Hashed | Hash index for equality queries on shard keys |
| Wildcard | Indexes all fields in a document (flexible schema support) |

---

## Replication: Replica Sets

A **Replica Set** consists of:
- One **primary** — accepts all writes
- One or more **secondaries** — asynchronously replicate the oplog from the primary
- Optional **arbiters** — vote in elections but hold no data

On primary failure, an automatic election promotes a secondary. Configurable write concerns:

| Write Concern | Behaviour |
|---------------|-----------|
| `w: 1` (default) | Acknowledge after primary write |
| `w: "majority"` | Acknowledge after majority of nodes write |
| `w: 0` | Fire-and-forget |

See [Replication](../../concepts/replication.md) for general theory.

---

## Sharding

MongoDB's built-in sharding routes queries via a **mongos** router process:

1. Choose a **shard key** (e.g., `user_id`).
2. Data is split into **chunks** and distributed across shards.
3. A **config server** replica set tracks chunk distribution.
4. `mongos` routes queries to the relevant shard(s).

Sharding strategies: range-based, hash-based, or zone-based. See [Sharding](../../concepts/sharding.md).

---

## Aggregation Pipeline

MongoDB's **Aggregation Pipeline** processes documents through a series of stages:

```javascript
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: { _id: "$customer_id", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } },
  { $limit: 10 }
])
```

Stages include: `$match`, `$group`, `$sort`, `$project`, `$lookup` (join), `$unwind`, `$facet`.

---

## ACID Transactions (v4.0+)

MongoDB added multi-document ACID transactions in v4.0 (replica sets) and v4.2 (sharded clusters):

```javascript
const session = client.startSession();
session.startTransaction();
try {
  await db.accounts.updateOne({ _id: "A" }, { $inc: { balance: -100 } }, { session });
  await db.accounts.updateOne({ _id: "B" }, { $inc: { balance: 100 } }, { session });
  await session.commitTransaction();
} catch (err) {
  await session.abortTransaction();
}
```

---

## Cross-References

| Topic | Document |
|-------|---------|
| Data model concepts | [Document Stores](document_stores.md) |
| Index internals | [B-Trees](../../concepts/b_trees.md) |
| Distributed guarantees | [CAP Theorem](../../concepts/cap_theorem.md) |
| Horizontal scaling | [Sharding](../../concepts/sharding.md) |
| High availability | [Replication](../../concepts/replication.md) |
| Transactional guarantees | [ACID Compliance](../../concepts/acid_compliance.md) |
| Cloud-hosted MongoDB | [Managed Cloud Databases](../infrastructure/managed_cloud_databases.md) |

---

## External Resources

- [MongoDB Official Documentation](https://www.mongodb.com/docs/)
- [MongoDB University — Free courses](https://learn.mongodb.com/)
- [WiredTiger storage engine documentation](https://source.wiredtiger.com/)
- [MongoDB Performance Best Practices](https://www.mongodb.com/docs/manual/administration/analyzing-mongodb-performance/)
