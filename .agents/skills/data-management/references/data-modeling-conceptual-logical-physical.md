# Data Modeling: Conceptual, Logical, and Physical Architecture

> **Mandate**: *Structure precedes optimization. Data modeling is the formal discipline of capturing business truth into unambiguous computational representations before disk layouts, query caches, or indexing hacks are introduced.*

---

## 1. The 3-Tier Modeling Separation

High-integrity data engineering separates data architecture into three distinct, non-leaking layers of abstraction:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        1. CONCEPTUAL MODEL                             │
│  Domain Entities, Business Rules, Bounded Contexts, Ubiquitous Lang    │
│  (Technology-agnostic, human-readable, domain truth)                   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Formalize & Normalize
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                         2. LOGICAL MODEL                               │
│  Attributes, Grain (G), Candidate Keys, Relational Normal Forms (3NF)  │
│  Integrity Constraints (CHECK, FK, UNIQUE, NOT NULL), Cardinalities    │
│  (Engine-neutral, formal relational/graph/document semantics)          │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Apply Workload Profile & Mechanics
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        3. PHYSICAL MODEL                               │
│  Storage Engine (B-Tree, LSM, Columnar), Physical Data Types, Layout   │
│  Indexes (B-Tree, Hash, GIN, HNSW), Partitioning, Sharding, Clustered  │
│  (Engine-specific, disk-aware, hardware-sympathetic performance)       │
└────────────────────────────────────────────────────────────────────────┘
```

### The Separation Law of Data Modeling
$$\text{Conceptual Truth} \xrightarrow{\quad\text{Formalize}\quad} \text{Logical Semantics} \xrightarrow{\quad\text{Profile \& Tune}\quad} \text{Physical Representation}$$

1. **Conceptual Layer**: What does the business mean by `Account`, `Order`, `InventoryItem`, or `Session`? What are the lifecycle states and boundary invariants?
2. **Logical Layer**: How are these entities related ($1:1$, $1:N$, $N:M$)? What is the primary grain ($\mathcal{G}$)? Which attributes depend strictly on the whole key (3NF/BCNF)? What integrity constraints must never be violated?
3. **Physical Layer**: How is this stored on disk or in memory? Is it a heap table with secondary B-trees (PostgreSQL), an index-organized table (MySQL InnoDB), a columnar projection (Parquet/ClickHouse), an append-only SSTable (Cassandra/RocksDB), or an adjacency list (Neo4j)?

---

## 2. Multi-Modal Grain & Identity Engineering

Every dataset—regardless of storage paradigm—must declare its **Grain** ($\mathcal{G}$): the atomic real-world event or entity that exactly one record represents.

$$\forall E \in \mathcal{E}, \quad \exists! \mathcal{G}(E) \implies \text{Identity}(E) \text{ is Immutable, Deterministic, and Collision-Resistant}$$

### 2.1 The Multi-Modal Identity Taxonomy

| Storage Paradigm | Grain Definition ($\mathcal{G}$) | Identity Strategy | Mechanical Characteristics |
| :--- | :--- | :--- | :--- |
| **Relational OLTP** | A discrete transactional state snapshot | Time-ordered UUIDv7, ULID, or natural composite key | Monotonic sequential clustering, prevents B-tree page fragmentation. |
| **Event Stream / Log** | A discrete immutable domain fact/event | Stream offset, partition + sequence ID, or SHA-256 payload hash | Append-only sequential write, monotonic ordering per partition. |
| **Document Store** | A self-contained aggregate boundary | Domain root ID (UUID / natural string key) | Co-locates child entities within document; single-record atomicity. |
| **Time-Series / IoT** | An atomic sensor or metric reading | Composite key: `(Entity_ID, Timestamp, Dimension_Hash)` | Chunked by time range; compressed columnar blocks; deduplication. |
| **Vector Index** | An embedding representation of a chunk | Source Entity URI + Chunk Hash + Vector Float Array | High-dimensional geometric point; nearest-neighbor distance metric. |
| **Graph Network** | An entity node or relationship edge | Global Node ID / Relationship Tuple `(Src_ID, Type, Dst_ID)` | Pointer adjacency; fast $O(1)$ edge-traversal index. |

### 2.2 Primary Key Strategy Trade-Off Matrix

Never choose primary key strategies on dogma. Evaluate structural trade-offs against physical storage mechanics:

```
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│ Key Type       │ Collision Safety │ Locality / B-Tree Sympathy │ Space Cost │ Leakage Risk   │
├────────────────┼──────────────────┼────────────────────────────┼────────────┼────────────────┤
│ Auto-Increment │ Machine-local    │ Excellent (Sequential)     │ 4-8 bytes  │ High (ID enum) │
│ UUIDv4         │ Global (Random)  │ Terrible (Page thrashing)  │ 16 bytes   │ Zero           │
│ UUIDv7 / ULID  │ Global (Timestamp│ Excellent (Sequential)     │ 16 bytes   │ Low (Time leak)│
│ Natural Key    │ Domain-dependent │ Variable                   │ Domain var │ Zero           │
│ Content Hash   │ Cryptographic    │ Uniform random dispersion  │ 32 bytes   │ Zero           │
└────────────────┴──────────────────┴────────────────────────────┴────────────┴────────────────┘
```

* **Default Recommendation for OLTP**: Use **UUIDv7** or **ULID**. They combine the global decentralization and collision safety of UUIDs with the sequential B-tree insertion locality of auto-incrementing integers, eliminating index page fragmentation.
* **Default Recommendation for Event Logs**: Use **Content-Addressed Hashes** ($H(\text{Payload})$) or **Monotonic Offsets** for deterministic deduplication.

---

## 3. Relational Normalization & The Denormalization Calculus

Normalization is not an academic chore; it is the formal mathematical method to eliminate update, insertion, and deletion anomalies.

### 3.1 The Normal Forms Condensed

* **First Normal Form (1NF)**: All attributes are atomic (no arrays/nested records in relational tuples); each record has a unique primary key.
* **Second Normal Form (2NF)**: 1NF + No partial dependencies (every non-key attribute depends on the *entire* candidate key, not a subset of a composite key).
* **Third Normal Form (3NF)**: 2NF + No transitive dependencies (non-key attributes depend *only* on candidate keys: $A \to B \to C$ is forbidden; extract $B \to C$).
* **Boyce-Codd Normal Form (BCNF)**: For every functional dependency $X \to Y$, $X$ must be a superkey.

### 3.2 The Intentional Denormalization Decision Calculus

Denormalization without measurement is premature corruption. Denormalization is **only permitted** when satisfying the formal trade-off equation:

$$\text{PermitDenormalization} \iff \left( \frac{\text{Read Volume}}{\text{Write Volume}} > 100 \right) \land \left( \text{Latency}_{\text{normalized\_join}} > \text{SLA}_{\text{read}} \right) \land \left( \text{Cost}_{\text{dual\_write\_sync}} \le \text{Risk}_{\text{inconsistency}} \right)$$

```
┌────────────────────────────────────────────────────────────────────────┐
│                     DENORMALIZATION GATEWAY                            │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Is the read-to-write ratio > 100:1?                                 │
│    NO  --> STOP. Keep schema normalized (3NF).                         │
│    YES --> Proceed to step 2.                                          │
│ 2. Has query profiling (EXPLAIN) proven that join latency breaks SLA?  │
│    NO  --> STOP. Add targeted index or optimize query.                 │
│    YES --> Proceed to step 3.                                          │
│ 3. Is there an automated synchronization mechanism?                   │
│    (Trigger, Transactional Outbox, Materialized View, or Sync Event)   │
│    NO  --> REJECT. Application drift will corrupt state.               │
│    YES --> APPROVE with explicit ADR documentation.                    │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Constraint-First Defense & Lowest Authoritative Layer

Application-layer validation is transient: background jobs, ETL imports, legacy scripts, and migration shims routinely bypass application code. **True domain invariants must be anchored at the lowest authoritative layer of the storage system.**

### 4.1 Invariant Layering Hierarchy

1. **Existence Invariant**: `NOT NULL` on every mandatory business attribute. Avoid using sentinel values (`-1`, `""`, `"N/A"`) to represent missing data.
2. **Uniqueness Invariant**: `UNIQUE` or composite unique constraints on natural business keys, even when surrogate keys exist.
3. **Referential Integrity**: `FOREIGN KEY` constraints with explicit referential actions (`ON DELETE RESTRICT` or `ON DELETE CASCADE`). Never rely on application logic alone to clean up child rows.
4. **Domain Range Invariants**: `CHECK` constraints enforcing business boundaries:
   ```sql
   CONSTRAINT check_order_total_positive CHECK (total_amount >= 0),
   CONSTRAINT check_percentage_bounds CHECK (discount_pct BETWEEN 0 AND 100),
   CONSTRAINT check_status_validity CHECK (status IN ('PENDING', 'PAID', 'SHIPPED', 'CANCELLED'))
   ```
5. **Multi-Column Exclusion Invariants**: Temporal overlaps, non-overlapping date ranges, or spatial boundaries (e.g. `EXCLUDE USING gist` in PostgreSQL).

### 4.2 When the Storage Engine Lacks Native Constraints (Lakehouses, Kafka, S3)
When using analytical storage (Parquet/S3), distributed NoSQL, or event streams where the engine does not enforce relational constraints:
* **The Gatekeeper Ingress Assertion**: Enforce strict schema validation (Protobuf, Avro, JSON Schema) at the write ingress.
* **Dead-Letter Quarantine**: Any payload violating constraints is routed to an unprocessable quarantine queue (`DLQ`) rather than polluting the lake.

---

## 5. Bounded Aggregates & Document Modeling (NoSQL Boundary Design)

In document and key-value databases, modeling is governed by **Aggregate Boundaries** (Domain-Driven Design):

* **Rule of Thumb**: *What is queried together and updated atomically belongs together in one document.*
* **1-to-Few ($N < 100$)**: Embed child entities directly in the parent document (e.g. `Order` embeds `OrderItems`).
* **1-to-Many / 1-to-Squillions ($N > 100$)**: Reference by identifier; store in separate collections/tables (e.g. `User` references `AuditLogs`, do not embed an unbounded log array inside a user document).
* **Avoid Unbounded Array Growth**: MongoDB and DynamoDB degrade severely when documents repeatedly reallocate disk space due to unbounded array appending.
