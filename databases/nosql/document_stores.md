# Document Stores

**Section:** [Databases](../index.md) › [NoSQL](index.md)

A **document store** is a type of NoSQL database that stores, retrieves, and manages semi-structured data as self-describing **documents** — typically JSON, BSON, or XML. Unlike relational databases, document stores do not enforce a fixed schema, allowing each document to have a different structure.

---

## Core Concepts

### Document

The fundamental unit of storage. A document is a self-contained data record, typically expressed as a JSON-like object:

```json
{
  "_id": "user_001",
  "name": "Alice",
  "email": "alice@example.com",
  "addresses": [
    { "type": "home", "city": "New York" },
    { "type": "work", "city": "San Francisco" }
  ],
  "preferences": { "theme": "dark", "notifications": true }
}
```

### Collection

A group of documents (analogous to a table in relational databases). Documents in the same collection need not share the same schema.

### Flexible Schema

Document stores allow **schema-on-read** — the schema is applied when data is queried, not when it is stored. This enables rapid iteration without `ALTER TABLE` migrations.

---

## Strengths

| Characteristic | Advantage |
|----------------|-----------|
| Flexible schema | Evolve data model without downtime |
| Nested/hierarchical data | No need for `JOIN` to retrieve related data stored in the same document |
| Horizontal scaling | [Sharding](../../concepts/sharding.md) is a first-class feature |
| Developer ergonomics | JSON maps naturally to application objects |

---

## Weaknesses

| Characteristic | Disadvantage |
|----------------|-------------|
| No multi-document ACID (historically) | Early document stores sacrificed [ACID compliance](../../concepts/acid_compliance.md) for performance |
| Duplicate data | Related data is denormalised; updates must propagate to multiple documents |
| Complex aggregations | Cross-collection queries are less efficient than SQL joins |
| No standard query language | Each system has its own query API |

---

## Storage Engines

Most document stores use either [B-Tree](../../concepts/b_trees.md) or [LSM Tree](../../concepts/lsm_trees.md) based storage engines:

| Database | Storage Engine | Structure |
|----------|---------------|-----------|
| MongoDB | WiredTiger | B-Tree (default) |
| CouchDB | B-Tree (custom) | B-Tree |
| Couchbase | Couchstore / Magma | B-Tree |
| Firestore | Bigtable-backed | LSM Tree |

---

## Cross-References

- [MongoDB](mongodb.md) — leading document store implementation
- [CAP Theorem](../../concepts/cap_theorem.md) — document stores often position as AP
- [Sharding](../../concepts/sharding.md) — native horizontal scaling in document stores
- [LSM Trees](../../concepts/lsm_trees.md) — write-optimised engine used by some implementations
- [B-Trees](../../concepts/b_trees.md) — WiredTiger's default storage structure

---

## External Resources

- [MongoDB Documentation — Documents](https://www.mongodb.com/docs/manual/core/document/)
- [CouchDB — The Definitive Guide](https://guide.couchdb.org/)
- [Firestore data model](https://firebase.google.com/docs/firestore/data-model)
