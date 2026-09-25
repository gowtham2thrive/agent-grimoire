# Multi-Archetype Data Modeling: The Rosetta Stone

> **Mandate**: *Principles remain invariant; physical representations adapt to access mechanics. This document demonstrates how the exact same business aggregate—**Product Catalog, Inventory, Customer Reviews & Semantic Search**—is modeled across 5 distinct storage archetypes without compromising data integrity.*

---

## The Canonical Domain Aggregate

```
                                  ┌────────────────────────┐
                                  │        PRODUCT         │
                                  │ ID: UUIDv7             │
                                  │ SKU: String (Unique)   │
                                  │ Title, Description     │
                                  │ BasePrice: Int (Cents) │
                                  └───────────┬────────────┘
                         ┌────────────────────┼────────────────────┐
                         │ 1:1                │ 1:N                │ 1:1
                         ▼                    ▼                    ▼
               ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
               │    INVENTORY     │  │     REVIEWS      │  │ VECTOR EMBEDDING │
               │ StockCount: Int  │  │ Rating: 1-5      │  │ Float[1536]      │
               │ WarehouseID      │  │ Body, AuthorID   │  │ ModelVersion     │
               └──────────────────┘  └──────────────────┘  └──────────────────┘
```

---

## Archetype 1: Relational OLTP (PostgreSQL 16)
* **Optimization Goal**: Strong ACID transactions, exact inventory balance, zero data anomalies.

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY, -- UUIDv7
    sku VARCHAR(64) NOT NULL UNIQUE,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    base_price_cents BIGINT NOT NULL,
    version INT NOT NULL DEFAULT 1,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT check_positive_price CHECK (base_price_cents > 0)
);

CREATE TABLE product_inventory (
    product_id UUID PRIMARY KEY REFERENCES products(id) ON DELETE CASCADE,
    stock_count INT NOT NULL DEFAULT 0,
    warehouse_id VARCHAR(32) NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT check_stock_non_negative CHECK (stock_count >= 0)
);

CREATE TABLE product_reviews (
    id UUID PRIMARY KEY,
    product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    customer_id UUID NOT NULL,
    rating SMALLINT NOT NULL,
    body TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT check_star_rating CHECK (rating BETWEEN 1 AND 5),
    CONSTRAINT one_review_per_customer UNIQUE (product_id, customer_id)
);
```

---

## Archetype 2: Document Store (MongoDB)
* **Optimization Goal**: Single-document read locality for e-commerce storefront rendering ($O(1)$ round-trip).

```javascript
// Bounded Aggregate Document with JSON-Schema validation
db.createCollection("products", {
   validator: {
      $jsonSchema: {
         bsonType: "object",
         required: ["_id", "sku", "title", "basePriceCents", "inventory"],
         properties: {
            _id: { bsonType: "string" }, // UUIDv7 string
            sku: { bsonType: "string" },
            title: { bsonType: "string" },
            basePriceCents: { bsonType: "int", minimum: 1 },
            inventory: {
               bsonType: "object",
               required: ["stockCount", "warehouseId"],
               properties: {
                  stockCount: { bsonType: "int", minimum: 0 },
                  warehouseId: { bsonType: "string" }
               }
            },
            // Reviews are high-velocity: embed only top 5 recent reviews; 
            // full reviews stored in separate collection to prevent unbounded document bloat
            recentReviews: {
               bsonType: "array",
               maximum: 5,
               items: {
                  bsonType: "object",
                  required: ["reviewId", "rating", "author"],
                  properties: {
                     rating: { bsonType: "int", minimum: 1, maximum: 5 }
                  }
               }
            }
         }
      }
   }
});
db.products.createIndex({ sku: 1 }, { unique: true });
```

---

## Archetype 3: Analytical Lakehouse (Apache Iceberg / Parquet)
* **Optimization Goal**: High-throughput columnar vectorization, partition pruning, historical time-travel.

```sql
CREATE TABLE lakehouse.catalog.products (
    product_id STRING,
    sku STRING,
    title STRING,
    category STRING,
    base_price_cents LONG,
    stock_count INT,
    avg_rating DOUBLE,
    total_reviews INT,
    event_timestamp TIMESTAMP
)
USING iceberg
PARTITIONED BY (category, days(event_timestamp))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd'
);
```

---

## Archetype 4: Event Stream (Apache Kafka / Avro Schema)
* **Optimization Goal**: Append-only auditability, decoupled asynchronous consumption, ordering per product.

```json
{
  "type": "record",
  "name": "ProductStockUpdatedEvent",
  "namespace": "com.inventory.events",
  "doc": "Emitted atomically when warehouse inventory level changes",
  "fields": [
    { "name": "eventId", "type": "string", "logicalType": "uuid" },
    { "name": "productId", "type": "string", "logicalType": "uuid" },
    { "name": "warehouseId", "type": "string" },
    { "name": "previousStock", "type": "int" },
    { "name": "newStock", "type": "int" },
    { "name": "occurredAt", "type": "long", "logicalType": "timestamp-millis" },
    { "name": "idempotencyKey", "type": "string" }
  ]
}
```
* **Kafka Key Strategy**: Partition by `productId` to guarantee strict causal ordering of stock updates per product.

---

## Archetype 5: High-Dimensional Vector Search (pgvector / Qdrant)
* **Optimization Goal**: Approximate Nearest Neighbor (ANN) semantic similarity for e-commerce search queries.

```sql
-- Using pgvector extension in PostgreSQL
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE product_embeddings (
    product_id UUID PRIMARY KEY REFERENCES products(id) ON DELETE CASCADE,
    embedding vector(1536), -- text-embedding-3-small dimension
    model_version VARCHAR(32) NOT NULL,
    indexed_text_hash CHAR(64) NOT NULL, -- Detects when description changes and re-index needed
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Fast Approximate Nearest Neighbor Index (HNSW with Cosine Distance)
CREATE INDEX idx_product_embeddings_hnsw 
ON product_embeddings 
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

---

## Comparison: Invariant Preservation Across Paradigms

| Invariant | Relational (PG) | Document (Mongo) | Lakehouse (Iceberg) | Event Log (Kafka) | Vector (HNSW) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Grain ($\mathcal{G}$)** | Row in `products` | Root document | Row in Parquet | Event message | Vector embedding point |
| **Identity** | UUIDv7 PK | `_id` string | `product_id` string | Partition key + Offset | `product_id` FK |
| **Constraints** | `CHECK`, `FK`, `UNIQUE` | JSON-Schema validator | Spark schema validation | Schema Registry (Avro) | Dimension constraint |
| **Evolution** | Expand-Contract | Additive optional fields | Schema evolution API | Full Avro Compatibility | Versioned model tag |
| **Idempotency** | ON CONFLICT DO UPDATE | Upsert (`$set`) | Merge into Iceberg table | Partition offset deduplication| Upsert vector by ID |
