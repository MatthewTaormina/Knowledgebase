# ACID Compliance

**Section:** [Concepts](index.md)

**ACID** is an acronym describing the four properties that guarantee reliable processing of database transactions. These properties ensure data integrity even in the presence of errors, power failures, and concurrent access.

---

## The Four Properties

### A — Atomicity

A transaction is **all-or-nothing**. Either all operations within the transaction succeed and are committed, or none of them are applied to the database.

**Implementation mechanisms:**
- **Undo logs / rollback segments:** If a transaction aborts mid-way, the database reverses all partial changes using a log of before-images.
- **Write-Ahead Logging (WAL):** Only committed transactions' changes are replayed on recovery.

**Example:** A bank transfer deducting $100 from account A and crediting $100 to account B must either complete both operations or neither.

### C — Consistency

A transaction brings the database from one **valid state** to another valid state. All data integrity constraints (primary keys, foreign keys, `CHECK` constraints, triggers) must hold before and after the transaction.

> Note: "Consistency" in ACID is an application-level guarantee, distinct from the "Consistency" in [CAP Theorem](cap_theorem.md), which refers to linearisability across distributed nodes.

### I — Isolation

Concurrently executing transactions appear to execute **serially** — each transaction is isolated from the intermediate states of others. The degree of isolation is configurable via **isolation levels**.

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Serialisation Anomaly |
|-----------------|-----------|---------------------|--------------|----------------------|
| Read Uncommitted | Possible | Possible | Possible | Possible |
| Read Committed | No | Possible | Possible | Possible |
| Repeatable Read | No | No | Possible | Possible |
| Serializable | No | No | No | No |

**Implementation mechanisms:**
- **MVCC (Multi-Version Concurrency Control):** Readers see a consistent snapshot; writers create new versions. Used by PostgreSQL, Oracle, MySQL InnoDB.
- **Two-Phase Locking (2PL):** Locks are acquired and held until transaction end. Used by older systems and some edge cases in modern engines.

### D — Durability

Once a transaction is committed, it **persists** even if the system crashes immediately afterwards.

**Implementation mechanisms:**
- **Write-Ahead Logging (WAL):** All changes are written to a sequential log on durable storage before being applied to data pages. On crash recovery, the WAL is replayed.
- **`fsync`:** Ensures the WAL has been flushed from OS buffers to physical storage before acknowledging the commit.
- **Battery-backed write caches:** RAID controllers with BBU can safely buffer writes without durability risk.

---

## ACID in PostgreSQL

PostgreSQL achieves full ACID compliance through:

| ACID Property | Mechanism |
|---------------|-----------|
| Atomicity | WAL + rollback |
| Consistency | Constraints + triggers + MVCC snapshot validation |
| Isolation | MVCC (Snapshot Isolation up to Repeatable Read; SSI for Serializable) |
| Durability | WAL + `fsync` + `synchronous_commit` setting |

See [PostgreSQL](../databases/relational/postgresql.md) for detailed implementation notes.

---

## ACID vs. BASE

NoSQL databases often relax ACID guarantees in favour of **BASE** semantics:

| Property | ACID | BASE |
|----------|------|------|
| State | Consistent after each transaction | Eventually Consistent |
| Availability | May sacrifice for consistency | High Availability prioritised |
| Trade-off | Correctness | Performance / scalability |

BASE: **B**asically **A**vailable, **S**oft state, **E**ventual consistency.

See [CAP Theorem](cap_theorem.md) for the distributed systems trade-off framework underpinning this distinction.

---

## Cross-References

- [CAP Theorem](cap_theorem.md) — distributed consistency trade-offs
- [Disk I/O](disk_io.md) — WAL and fsync I/O mechanics
- [PostgreSQL](../databases/relational/postgresql.md) — full ACID implementation example
- [Sharding](sharding.md) — how distributed transactions complicate ACID
- [Replication](replication.md) — durability and consistency across replicas

---

## External Resources

- [Gray & Reuter (1992) — "Transaction Processing: Concepts and Techniques"](https://www.elsevier.com/books/transaction-processing/gray/978-0-08-051414-4)
- [PostgreSQL — Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [Designing Data-Intensive Applications — Chapter 7, Martin Kleppmann](https://dataintensive.net/)
