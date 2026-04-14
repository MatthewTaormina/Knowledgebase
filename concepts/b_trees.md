# B-Trees

**Section:** [Concepts](index.md)

A **B-Tree** (Balanced Tree) is a self-balancing, ordered tree data structure designed to store large amounts of sorted data and support efficient search, sequential access, insertion, and deletion operations. It is the dominant index structure in relational database management systems (RDBMSs) and many other storage engines.

---

## Background & Motivation

Traditional binary search trees degrade to O(n) performance in the worst case (skewed trees). Red-Black trees and AVL trees improve on this but are optimised for in-memory use with pointer-heavy nodes. Databases, however, must minimise **disk I/O** — each node read is a page fetch costing orders of magnitude more than an in-memory comparison.

B-Trees were invented by Rudolf Bayer and Edward McCreight at Boeing Research Labs (1970) specifically to address this: by making each node wide (holding many keys), the tree height stays small and the number of page reads needed to traverse from root to leaf is minimised.

See also: [Disk I/O](disk_io.md) for the hardware constraints that motivate B-Tree design.

---

## Formal Definition

A B-Tree of order **t** (minimum degree) satisfies:

1. **Node capacity:** Every node except the root has at least `t − 1` keys and at most `2t − 1` keys. The root has at least 1 key.
2. **Child count:** An internal node with `k` keys has exactly `k + 1` children.
3. **Sorted keys:** Keys within each node are stored in non-decreasing order.
4. **Balanced:** All leaf nodes reside at the same depth.
5. **Key-separator property:** For an internal node key `k_i`, all keys in the subtree rooted at child `c_i` are less than `k_i`, and all keys in the subtree rooted at child `c_{i+1}` are greater than `k_i`.

---

## B-Tree vs. B+ Tree

Most databases implement the **B+ Tree** variant rather than the classic B-Tree:

| Property | B-Tree | B+ Tree |
|----------|--------|---------|
| Data storage | Keys and data in internal AND leaf nodes | Data pointers only in **leaf nodes** |
| Leaf node links | No sibling pointers | Leaves form a **doubly-linked list** |
| Range scans | Requires tree traversal | Efficient — traverse leaf chain |
| Internal node capacity | Lower (stores data) | Higher (stores keys only) |
| Search | May terminate early at internal node | Always reaches a leaf |

The leaf-linked-list property of B+ Trees makes **range queries** (`BETWEEN`, `ORDER BY`, `<`, `>`) highly efficient — a critical requirement for SQL workloads.

> **PostgreSQL**, MySQL InnoDB, SQLite, and Oracle all use B+ Trees for their default index type.

---

## Time Complexity

| Operation | Average | Worst Case |
|-----------|---------|------------|
| Search | O(log n) | O(log n) |
| Insert | O(log n) | O(log n) |
| Delete | O(log n) | O(log n) |
| Range scan of k items | O(log n + k) | O(log n + k) |

Height of a B-Tree with `n` keys and minimum degree `t`:

```
h ≤ log_t ⌈(n+1)/2⌉
```

For `t = 100` and `n = 1,000,000` keys, `h ≤ 3`. This means at most **3 page reads** to locate any key.

---

## Node Structure (On-Disk)

A B+ Tree node maps to a single **page** (typically 4 KB–16 KB on disk). The layout of an internal node:

```
┌─────────────────────────────────────────────────────────────┐
│ Page Header │ P0 │ K1 │ P1 │ K2 │ P2 │ … │ K(n) │ P(n) │  │
└─────────────────────────────────────────────────────────────┘
  P = child page pointer,  K = separator key
```

A leaf node:

```
┌──────────────────────────────────────────────────────────────────────┐
│ Page Header │ K1 │ Ptr1 │ K2 │ Ptr2 │ … │ K(n) │ Ptr(n) │ Next Page │
└──────────────────────────────────────────────────────────────────────┘
  Ptr = heap tuple pointer (TID in PostgreSQL),  Next Page = right sibling
```

---

## Insertion

1. Search for the leaf where the key belongs.
2. If the leaf has room (`< 2t − 1` keys), insert in sorted order.
3. If the leaf is **full**, **split** it at the median key:
   - Left child: lower half of keys.
   - Right child: upper half of keys.
   - Median key is **promoted** to the parent.
4. Repeat split upward if the parent is also full. If the root splits, tree height increases by 1.

---

## Deletion

1. Locate the key.
2. If in a leaf with enough keys (> t − 1): remove directly.
3. If in an internal node: replace with **in-order predecessor** (max key in left subtree) or **in-order successor** and delete that from the leaf.
4. If a node falls below minimum occupancy after deletion, **rebalance**:
   - **Borrow** a key from a sibling (rotation), or
   - **Merge** with a sibling — the separator key from the parent descends.

---

## Page Splits and Write Amplification

Splits and merges can propagate up the tree, causing multiple page writes per logical insert/delete. This is one of the motivations for the alternative [LSM Tree](lsm_trees.md) structure, which avoids in-place updates entirely.

---

## Fill Factor & Fragmentation

Databases allow tuning of the **fill factor** — the fraction of each page filled during initial construction:

- A fill factor of **100%** maximises space efficiency but causes immediate splits on the next insert.
- A fill factor of **70–90%** leaves room for future inserts, reducing split frequency.

Over time, deletions leave pages partially empty (fragmentation), requiring periodic **VACUUM** (PostgreSQL) or `OPTIMIZE TABLE` (MySQL) operations.

---

## Real-World Usage in Databases

| Database | Index Type | Notes |
|----------|------------|-------|
| PostgreSQL | B+ Tree (default) | Supports unique, partial, covering, and multi-column indexes |
| MySQL InnoDB | B+ Tree | Clustered primary key; secondary indexes store PK value |
| SQLite | B+ Tree | Entire database stored in a single B-Tree file |
| Oracle | B+ Tree | Also supports Reverse Key and Index-Organised Tables (IOT) |
| SQL Server | B+ Tree | Clustered and non-clustered indexes |

See [PostgreSQL](../databases/relational/postgresql.md) for a detailed example of B-Tree usage in a real system.

---

## Comparison with Alternative Index Structures

| Structure | Read Performance | Write Performance | Best Use Case |
|-----------|-----------------|-------------------|---------------|
| B+ Tree | Excellent (low height) | Good (in-place updates) | OLTP, range queries |
| [LSM Tree](lsm_trees.md) | Good (compaction overhead) | Excellent (sequential writes) | Write-heavy workloads |
| Hash Index | O(1) point lookup | O(1) insert | Exact-match lookups only |
| Bitmap Index | Fast for low-cardinality | Expensive to update | OLAP / data warehousing |

See [Indexing Strategies](indexing_strategies.md) for a full comparison.

---

## External Resources

- [Bayer & McCreight (1970) — "Organization and Maintenance of Large Ordered Indexes"](https://link.springer.com/article/10.1007/BF00288683) — original B-Tree paper
- [PostgreSQL documentation — Index Types](https://www.postgresql.org/docs/current/indexes-types.html)
- [CMU 15-445 Database Systems — B+ Trees lecture](https://15445.courses.cs.cmu.edu/fall2023/slides/07-trees.pdf)
- [MySQL InnoDB B-Tree Index documentation](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html)
