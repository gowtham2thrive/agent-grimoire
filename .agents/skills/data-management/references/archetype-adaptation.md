# Universal Archetype Adaptation Matrix (Data Management)

The 8 invariants adapt dynamically across every storage archetype by mapping to mechanical storage realities:

| Archetype | Storage Engine Model | Primary Consistency & Locks | Schema Evolution Mode |
| :--- | :--- | :--- | :--- |
| **Relational OLTP**<br>*(Postgres, MySQL, Oracle)* | B-Tree / Heap; in-place row mutation | ACID, MVCC, Serial / Read-Committed, page latches | Online DDL, Expand-Contract, lock-free indexes |
| **Document / Key-Value**<br>*(MongoDB, DynamoDB)* | B-Tree / LSM; denormalized JSON aggregates | Single-document ACID; tunable read/write quorums | Read-time schema, dual-write backfill, additive fields |
| **Analytical Lakehouse**<br>*(Snowflake, DuckDB, Iceberg)* | Columnar vectorization (Parquet/ORC); dictionary encoding | Append-mostly; ACID via Snapshot / Time-travel metadata | Additive schema evolution, partition pruning |
| **Event Streams**<br>*(Kafka, Redpanda, Pulsar)* | Append-only partitioned commit logs; sequential disk I/O | Strict ordering per partition; at-least-once delivery | Schema Registry (Avro/Protobuf), Full compatibility |
| **Vector Search**<br>*(pgvector, Qdrant, Milvus)* | HNSW graph / Inverted vector index | Eventual consistency; high-dimensional distance metrics | Parallel re-indexing, zero-downtime pointer swap |
| **Graph Network**<br>*(Neo4j, Memgraph)* | Adjacency index; index-free pointer chasing | Graph traversal consistency; node/edge constraints | Monotonic relationship label & property expansion |
| **Embedded & Edge**<br>*(SQLite, DuckDB, IndexedDB)* | Single-file B-Tree; memory-mapped WAL | Single-writer process lock; crash-safe WAL journal | Embedded migration hooks, `user_version` pragma |
| **AI Agent State**<br>*(Cognitive Memory, Sessions)* | Structured JSON state, Vector episodic log, KV scratchpad | Causal turn/session ordering; non-blocking snapshotting | Checkpoint versioning, rolling context window TTL |
| **Mobile / Offline-First**<br>*(Client sync engines)* | Local embedded SQLite with sync queue | Eventual consistency; local-first optimistic writes | Delta sync protocol, CRDTs or monotonic vector clocks |
| **Immutable Ledgers**<br>*(WORM, Audit trails)* | Cryptographic Merkle DAG; append-only hash chains | Absolute immutability; tamper-evident verification | Cryptographic Shredding for privacy compliance |
