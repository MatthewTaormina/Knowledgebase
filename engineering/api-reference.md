# API Reference

**Section:** [Engineering](index.md)

This document catalogues all public API endpoints, their request/response schemas, and usage examples.

---

## Overview

The API Gateway (described in the [Architecture Overview](architecture-overview.md)) exposes two interfaces:

| Interface | Base URL |
|-----------|---------|
| REST | `/api/v1/` |
| GraphQL | `/graphql` |

All requests must include a valid **Bearer token** in the `Authorization` header.

---

## REST Endpoints

### `GET /api/v1/items`

Returns a paginated list of domain items.

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: `1`) |
| `limit` | integer | No | Items per page (default: `20`, max: `100`) |
| `filter` | string | No | Free-text search filter |

**Response — 200 OK**

```json
{
  "data": [
    { "id": "abc123", "name": "Example Item", "createdAt": "2024-01-15T10:00:00Z" }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1
  }
}
```

---

### `POST /api/v1/items`

Creates a new domain item.

**Request Body**

```json
{
  "name": "New Item",
  "description": "Optional description text."
}
```

**Response — 201 Created**

```json
{
  "id": "xyz789",
  "name": "New Item",
  "description": "Optional description text.",
  "createdAt": "2024-01-16T08:30:00Z"
}
```

---

### `GET /api/v1/items/{id}`

Returns a single item by its unique identifier.

**Path Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Unique item identifier |

**Response — 200 OK**

```json
{
  "id": "abc123",
  "name": "Example Item",
  "description": "...",
  "createdAt": "2024-01-15T10:00:00Z"
}
```

**Response — 404 Not Found**

```json
{ "error": "Item not found." }
```

---

## GraphQL API

The GraphQL schema mirrors the REST resources. Explore and execute queries interactively via the built-in **GraphiQL** playground available at `/graphql` when running in non-production environments.

**Example query**

```graphql
query GetItem($id: ID!) {
  item(id: $id) {
    id
    name
    description
    createdAt
  }
}
```

---

## System Diagram

The components that handle these requests are illustrated in the [Architecture Overview](architecture-overview.md).

![Architecture Diagram](../assets/architecture-diagram.svg)

---

## Related Data Schemas

The field definitions used in request/response bodies are derived from the schemas documented in the [Sample Dataset](../datasets/sample-dataset.md).

---

## External Resources

- [OpenAPI Specification (OAS 3.0)](https://swagger.io/specification/) — standard used to describe these REST endpoints
- [GraphQL specification](https://spec.graphql.org/) — formal specification for the GraphQL interface
- [JWT introduction](https://jwt.io/introduction) — explanation of the Bearer token format used for authentication
