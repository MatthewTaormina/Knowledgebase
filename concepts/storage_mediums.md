# Storage Mediums

**Section:** [Concepts](index.md)

This document compares the primary storage technologies used in database infrastructure: Hard Disk Drives (HDDs), SATA Solid-State Drives (SSDs), NVMe SSDs, and emerging Persistent Memory (PMEM).

---

## Technology Comparison

| Property | HDD | SATA SSD | NVMe SSD | Optane PMEM |
|----------|-----|----------|----------|-------------|
| Technology | Spinning magnetic platters | NAND Flash (SATA interface) | NAND Flash (PCIe interface) | 3D XPoint / NAND |
| Sequential Read | 100–250 MB/s | 400–560 MB/s | 3–7 GB/s | 6–13 GB/s |
| Sequential Write | 80–160 MB/s | 300–530 MB/s | 1–7 GB/s | 2–8 GB/s |
| Random Read (4K IOPS) | 75–200 IOPS | 10K–100K IOPS | 100K–1M IOPS | 10M+ IOPS |
| Latency | 5–15 ms | 0.1–0.5 ms | 0.02–0.1 ms | ~0.3 µs |
| Durability (TBW) | Very high | Moderate (TLC) – High (MLC) | Moderate–High | High |
| Cost per GB (2024) | ~$0.02 | ~$0.08 | ~$0.10 | ~$5+ |
| Capacity (2024) | Up to 20 TB | Up to 4 TB (consumer) | Up to 8 TB | 128–512 GB (DIMM) |

---

## HDD (Hard Disk Drive)

HDDs store data on spinning magnetic platters read by an actuator arm. The mechanical nature introduces **seek time** (arm positioning) and **rotational latency** (waiting for the sector to rotate under the head), making random I/O expensive.

**Database use cases:** Cold data archival, large sequential workloads (backups, bulk exports), cost-sensitive deployments where latency is not critical.

---

## SATA SSD

NAND Flash-based drives connected over the legacy SATA interface. Eliminates mechanical latency but the SATA bus caps throughput at ~600 MB/s.

**Database use cases:** General-purpose production databases where budget constraints prevent NVMe adoption; good replacement for HDDs in existing SATA bays.

---

## NVMe SSD

Connects directly to CPU via **PCIe lanes**, bypassing SATA's legacy command queue bottleneck (32 commands) in favour of NVMe's 65,535 queues × 65,535 commands. Dramatically improves IOPS and reduces latency.

**Database use cases:** Primary storage for high-throughput OLTP databases; WAL device for PostgreSQL/MySQL; any workload requiring >50K IOPS.

### NAND Flash Types

| Type | Bits/Cell | Endurance | Speed | Cost |
|------|-----------|-----------|-------|------|
| SLC | 1 | Very High (~100K P/E) | Fastest | Very High |
| MLC | 2 | High (~10K P/E) | Fast | High |
| TLC | 3 | Moderate (~3K P/E) | Good | Medium |
| QLC | 4 | Lower (~1K P/E) | Slower | Low |

---

## Persistent Memory (PMEM / NVDIMM)

Sits in DRAM slots but retains data across power cycles. Accessible at byte granularity (unlike block devices), enabling new database architectures where WAL can be written to PMEM at memory speeds.

**Database use cases:** WAL acceleration; in-memory databases requiring persistence without full DRAM cost.

---

## Cross-References

- [Memory Hierarchy](memory_hierarchy.md) — where each medium fits in the performance pyramid
- [Disk I/O](disk_io.md) — I/O patterns that influence medium selection
- [B-Trees](b_trees.md) — how page size aligns to storage block size

---

## External Resources

- [Flash Memory Guide — AnandTech](https://www.anandtech.com/show/9469/the-nand-flash-guide)
- [NVMe specification](https://nvmexpress.org/developers/nvme-specification/)
- [Intel Optane PMEM product brief](https://www.intel.com/content/www/us/en/products/docs/memory-storage/optane-persistent-memory/overview.html)
