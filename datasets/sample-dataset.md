# Sample Dataset

**Section:** [Datasets](index.md)

This document demonstrates the dataset document format: a combination of metadata, a tabular data section, and cross-references to relevant engineering documentation.

---

## Dataset Metadata

| Field | Value |
|-------|-------|
| **Name** | `items` |
| **Version** | `1.0.0` |
| **Owner** | Engineering |
| **Last Updated** | 2024-01-16 |
| **Row Count** | 3 (sample) |

---

## Schema

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| `id` | `string (UUID)` | No | Unique identifier for the item |
| `name` | `string` | No | Human-readable item name (max 255 chars) |
| `description` | `string` | Yes | Optional long-form description |
| `createdAt` | `datetime (ISO 8601)` | No | Timestamp of record creation |
| `updatedAt` | `datetime (ISO 8601)` | Yes | Timestamp of last modification |

---

## Sample Rows

| id | name | description | createdAt | updatedAt |
|----|------|-------------|-----------|-----------|
| `abc123` | Alpha Item | First sample record | `2024-01-10T09:00:00Z` | `2024-01-12T14:30:00Z` |
| `def456` | Beta Item | Second sample record | `2024-01-11T10:15:00Z` | _(null)_ |
| `xyz789` | Gamma Item | Third sample record | `2024-01-16T08:30:00Z` | _(null)_ |

---

## Cross-References

- The API endpoints that expose this dataset → [API Reference](../engineering/api-reference.md)
- System components that persist this data → [Architecture Overview](../engineering/architecture-overview.md)

---

## External Resources

- [ISO 8601 date/time standard](https://www.iso.org/iso-8601-date-and-time-format.html) — format used for all timestamp columns
- [UUID specification (RFC 4122)](https://datatracker.ietf.org/doc/html/rfc4122) — format used for the `id` column
