# Zero-Downtime Migrations & Schema Evolution Protocol

> **Mandate**: *State outlives application code. Deployments are transient; database migrations alter the living memory of the business. Zero-downtime schema evolution ensures that shared state transitions smoothly without table locks, application crashes, or data loss across distributed consumers.*

---

## 1. The 3-Phase Expand-Contract Protocol

Any breaking schema modification—renaming a column, changing a data type, splitting a table, or altering nullability—must **never** occur in a single atomic release when active production consumers exist ($N_{\text{consumers}} > 1$). 

The system must execute across three distinct lifecycle phases:

```mermaid
flowchart TD
    subgraph Phase 1: Expand
        A1["1. Add New Column/Table (Nullable or with Safe Default)"] --> A2["2. Deploy App V1: Writes to BOTH old & new; Reads from old"]
    end
    
    subgraph Phase 2: Migrate
        B1["3. Execute Background Chunked Backfill (DML)"] --> B2["4. Verify Parity & Catch Up In-Flight Writes"]
        B2 --> B3["5. Deploy App V2: Writes to BOTH old & new; Reads from NEW"]
    end
    
    subgraph Phase 3: Contract
        C1["6. Verify Zero Traffic / Null Read-Load on Old Column"] --> C2["7. Deploy App V3: Writes & Reads ONLY from NEW"]
        C2 --> C3["8. Drop Old Column/Table (Lock-Free DDL)"]
    end

    Phase 1: Expand --> Phase 2: Migrate --> Phase 3: Contract
```

### Mathematical Invariance Rule
$$\Delta_{\text{breakage}} = 0 \quad \forall t \in [t_{\text{deploy\_start}}, t_{\text{all\_consumers\_updated}}]$$
At every instant $t$, an active deployment of application version $V_{k}$ and version $V_{k+1}$ must both run concurrently against the database without syntax errors, missing columns, or null pointer exceptions.

---

## 2. DDL Locking Mechanics & Lock-Free Execution

Database engines acquire locks during schema modifications. A naive `ALTER TABLE` can lock an entire table against reads and writes (`ACCESS EXCLUSIVE`), cascading into thread exhaustion and cascading system outages.

### 2.1 The Relational DDL Safety Matrix

| Operation | PostgreSQL Risk & Mitigation | MySQL (InnoDB) Risk & Mitigation |
| :--- | :--- | :--- |
| **Add Column without default** | Metadata only ($O(1)$ lock). Safe. | Metadata only in 8.0+ ($O(1)$). Safe. |
| **Add Column with default** | Constant default is $O(1)$ in Postgres 11+. Volatile default (`NOW()`, `RANDOM()`) rewrites entire table! Use constant or backfill. | `ALGORITHM=INSTANT` in MySQL 8.0.12+. Earlier versions rewrite table. |
| **Add Index** | **NEVER** use `CREATE INDEX` on live tables. Always use: `CREATE INDEX CONCURRENTLY`. | Use `ALGORITHM=INPLACE, LOCK=NONE`. |
| **Rename Column** | Breaks running application versions immediately. Must use Expand-Contract. | Breaks running application versions immediately. Must use Expand-Contract. |
| **Change Data Type** | Full table rewrite and exclusive lock! Must use Expand-Contract with backfill. | Full table rewrite in most cases. Must use Expand-Contract with backfill. |
| **Add NOT NULL** | Scanning table locks writers. Mitigation: Add `CHECK (col IS NOT NULL) NOT VALID;` then validate asynchronously. | Rewrites table unless done via generated column or strict mode checks. |
| **Drop Column** | Safe in metadata, but breaks older app versions still referencing it in `SELECT *`. | Safe with `ALGORITHM=INSTANT`, but breaks older app code. Contract phase only. |

### 2.2 The Lock Timeout Rule
Every migration script executed in production must set an explicit, strict lock timeout before running DDL:

```sql
-- PostgreSQL Lock Safety Guard
SET lock_timeout = '3s';
SET statement_timeout = '10min';

-- If lock cannot be acquired within 3 seconds, abort and retry
-- prevents queuing behind slow queries and starving all connections
ALTER TABLE orders ADD COLUMN status_v2 VARCHAR(50);
```

---

## 3. Separation of Structural DDL from Transactional Backfill DML

A catastrophic failure mode is bundling table schema alterations (DDL) and million-row data updates (DML) into a single database transaction.

$$\text{Migration Unit} \neq \text{DDL} \cup \text{Massive DML}$$

### The Cardinal Rule
1. **Migration Step 1 (DDL)**: Fast, non-blocking structural change (sub-second lock acquisition). Commit immediately.
2. **Migration Step 2 (DML Backfill)**: Run out-of-band as a chunked background script or streaming worker.
3. **Migration Step 3 (Validation DDL)**: Add constraints or drop legacy artifacts once backfill is 100% complete.

### Chunked Background Backfill Algorithm
Never run `UPDATE users SET full_name = first_name || ' ' || last_name;` across 10,000,000 rows in one transaction (causes massive undo/redo log growth, lock escalation, and replication lag).

**Use Keyset Batching**:
```sql
-- Safe chunked backfill pattern (batch size = 5000)
DO $$
DECLARE
    batch_size INT := 5000;
    last_id BIGINT := 0;
    rows_updated INT;
BEGIN
    LOOP
        UPDATE users
        SET full_name = first_name || ' ' || last_name
        WHERE id > last_id AND id <= last_id + batch_size AND full_name IS NULL;
        
        GET DIAGNOSTICS rows_updated = ROW_COUNT;
        EXIT WHEN rows_updated = 0;
        
        last_id := last_id + batch_size;
        COMMIT; -- Releases locks and flushes transaction logs per chunk
        PERFORM pg_sleep(0.05); -- Yields CPU and replication bandwidth
    END LOOP;
END $$;
```

---

## 4. Handling Dirty Brownfield Data: Dual-Horizon Constraints

When applying a new constraint (`NOT NULL` or `CHECK`) to an existing production table with millions of legacy rows, historical records often violate the new business rule.

```
Existing Legacy Rows (Dirty / Unverified)        New Incoming Rows (Strict)
[ Row 1: status = NULL                  ]        [ Row 10001: status = 'ACTIVE' ]
[ Row 2: status = 'LEGACY_ERR'          ]  --->  [ Row 10002: status = 'ACTIVE' ]
[ Row 3: status = NULL                  ]        [ Row 10003: status = 'PENDING']
         │                                                │
         ▼                                                ▼
  Background Remediation Worker                   Enforced via NOT VALID
```

### The 2-Step Constraint Validation Protocol (PostgreSQL Specimen)

**Step 1: Enforce on NEW writes immediately without checking legacy rows ($O(1)$ lock)**
```sql
ALTER TABLE accounts 
ADD CONSTRAINT check_balance_non_negative 
CHECK (balance >= 0) NOT VALID;
```
*Result*: New inserts and updates must satisfy `balance >= 0`. Existing rows are not scanned, avoiding long table locks.

**Step 2: Validate existing rows in background without blocking writers**
```sql
ALTER TABLE accounts 
VALIDATE CONSTRAINT check_balance_non_negative;
```
*Result*: Scans historical rows with a share lock (reads and writes continue uninterrupted). If historical violations exist, clean them up via background remediation before running `VALIDATE CONSTRAINT`.

---

## 5. Reversible Rollbacks vs. Forward-Only Evolution

### 5.1 The Downward Migration Myth
In distributed architectures, rolling back a schema migration by running a `down()` script often causes catastrophic data corruption:
* If App V2 wrote data into new column $C_2$, running `ALTER TABLE DROP COLUMN C_2` destroys live customer data written during the release window!

### 5.2 The Forward-Only Discipline
1. **Schema migrations must be strictly additive and forward-only**.
2. If a migration is defective, roll back the **application code** to Version 1 (which still works because of the Expand phase).
3. Repair the defect and deploy a **new forward migration** ($V_{k+2}$), never an unverified downward drop of living columns.

---

## 6. The Phase-Aware Migration Boundary

Not every environment requires the overhead of Expand-Contract:

* **Production with Active Consumers ($N_{\text{consumers}} > 1$, SLA $\ge 99.9\%$)**: Expand-Contract is **mandatory**. Zero tolerance for downtime or locking DDL.
* **Greenfield / Prototype Mode ($N_{\text{consumers}} \le 1$, $t < t_{\text{production}}$)**: Drop tables, rename columns in-place, and recreate schemas with zero ceremony. Do not burden rapid early prototyping with multi-phase expansion.
