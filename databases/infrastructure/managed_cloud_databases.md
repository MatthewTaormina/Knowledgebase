# Managed Cloud Databases

**Section:** [Databases](../index.md) › [Infrastructure](index.md)

**Managed cloud databases** (Database-as-a-Service, DBaaS) are database instances operated and maintained by a cloud provider. The provider handles provisioning, patching, backups, monitoring, and failover — allowing teams to focus on application development rather than database operations.

---

## Why Managed Databases

| Responsibility | Self-Managed | Managed (DBaaS) |
|----------------|-------------|-----------------|
| Hardware provisioning | Manual | Provider |
| OS patching | Manual | Provider |
| Database version upgrades | Manual | Automated / assisted |
| Automated backups | Manual | Provider (point-in-time restore) |
| High availability / failover | Manual setup | Built-in |
| Monitoring & alerting | Manual setup | Basic built-in; advanced optional |
| Scaling | Manual | Vertical and/or horizontal (auto-scaling) |

---

## Major Providers

### Amazon Web Services (AWS)

| Service | Underlying Engine | Notes |
|---------|------------------|-------|
| **Amazon RDS** | PostgreSQL, MySQL, MariaDB, Oracle, SQL Server | Multi-AZ failover, read replicas |
| **Amazon Aurora** | MySQL-compatible / PostgreSQL-compatible | 6-way replicated storage; up to 15 read replicas; serverless v2 |
| **Amazon DynamoDB** | Proprietary key-value / document | Serverless, auto-scaling, global tables |
| **Amazon ElastiCache** | Redis / Memcached | Managed in-memory caching |
| **Amazon Redshift** | Columnar data warehouse | Petabyte-scale analytics |
| **Amazon DocumentDB** | MongoDB-compatible (proprietary) | Atlas-compatible API |

### Google Cloud Platform (GCP)

| Service | Underlying Engine | Notes |
|---------|------------------|-------|
| **Cloud SQL** | PostgreSQL, MySQL, SQL Server | Managed relational DB |
| **Cloud Spanner** | Proprietary (TrueTime / Paxos) | Globally distributed, ACID, SQL |
| **Firestore** | NoSQL document store | Serverless, offline-capable |
| **Bigtable** | Wide-column (LSM-based) | Petabyte-scale, low-latency |
| **BigQuery** | Serverless columnar data warehouse | Pay-per-query |
| **Memorystore** | Redis / Memcached | Managed in-memory |

### Microsoft Azure

| Service | Underlying Engine | Notes |
|---------|------------------|-------|
| **Azure SQL Database** | SQL Server | Fully managed, serverless tier available |
| **Azure Database for PostgreSQL** | PostgreSQL | Flexible Server with availability zones |
| **Azure Cosmos DB** | Multi-model (document, graph, table, Cassandra API) | Global distribution, 99.999% SLA |
| **Azure Cache for Redis** | Redis | Managed Redis with Enterprise tier |

---

## Key Managed Database Concepts

### Multi-AZ and High Availability

Most managed services offer **Multi-AZ** deployments: a standby replica in a different availability zone is kept in sync. On primary failure, automatic failover occurs in 30–60 seconds.

See [Replication](../../concepts/replication.md) for the underlying theory.

### Point-in-Time Recovery (PITR)

Managed services continuously archive transaction logs and take periodic snapshots. PITR allows restoring the database to any point within the retention window (typically 7–35 days).

This relies on WAL/binlog-based replication — see [ACID Compliance](../../concepts/acid_compliance.md) for WAL durability concepts.

### Read Replicas

Cloud providers support adding read replicas with a single API call. Traffic can be directed to replicas for read-heavy workloads, offloading the primary.

### Serverless Databases

**Serverless** databases (Aurora Serverless v2, Neon, PlanetScale) scale compute capacity automatically to zero when idle and scale up to handle bursts, billed per request or per ACU (Aurora Capacity Unit).

---

## Trade-offs vs. Self-Managed

| Factor | Managed | Self-Managed |
|--------|---------|-------------|
| Operational overhead | Low | High |
| Cost at scale | Higher (vendor premium) | Lower (hardware/cloud VMs) |
| Customisation | Limited | Full control |
| Vendor lock-in | Potential | None |
| Compliance | Provider certifications (SOC 2, HIPAA, etc.) | Self-certified |

---

## Cross-References

- [Replication](../../concepts/replication.md) — Multi-AZ, read replicas, failover
- [Sharding](../../concepts/sharding.md) — horizontal scaling in Aurora, Cosmos DB, Spanner
- [CAP Theorem](../../concepts/cap_theorem.md) — Cloud Spanner aims for external consistency (beyond CP)
- [Database Clustering](database_clustering.md) — self-managed alternative
- [PostgreSQL](../relational/postgresql.md) — available on RDS, Cloud SQL, Azure Database
- [Redis](../in_memory/redis.md) — available on ElastiCache, Memorystore, Azure Cache

---

## External Resources

- [AWS RDS Documentation](https://docs.aws.amazon.com/rds/)
- [Amazon Aurora Technical Overview](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.html)
- [Google Cloud Spanner — TrueTime and External Consistency](https://cloud.google.com/spanner/docs/true-time-external-consistency)
- [Azure Cosmos DB — Global Distribution](https://learn.microsoft.com/en-us/azure/cosmos-db/distribute-data-globally)
