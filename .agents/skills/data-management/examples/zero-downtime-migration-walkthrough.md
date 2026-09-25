# Zero-Downtime Migration Walkthrough: Production Column Split

> **Scenario**: *A high-traffic SaaS production database has a `users` table with 15,000,000 rows. The legacy schema stores a single unstructured `full_name VARCHAR(255)` column. Business requirements demand splitting this into `first_name VARCHAR(100) NOT NULL` and `last_name VARCHAR(100) NOT NULL`.*
> 
> *Directly executing `ALTER TABLE users ADD COLUMN first_name ... NOT NULL;` would lock the table for minutes, causing connection pool exhaustion and global downtime. This walkthrough shows the zero-downtime Expand-Contract execution.*

---

## Timeline & Architecture Overview

```
Week 1 (Day 1)          Week 1 (Day 2)             Week 2 (Day 4)             Week 3 (Day 7)
Phase 1: EXPAND         Phase 2A: BACKFILL         Phase 2B: DUAL-READ        Phase 3: CONTRACT
─────────────────────────────────────────────────────────────────────────────────────────────────
• Add nullable cols     • Run chunked background   • Deploy App V2:           • Deploy App V3:
  with lock timeout       backfill worker            Reads from first/last      Writes & reads only
• Deploy App V1:        • Validate 100% parity     • Add NOT VALID check      • Drop old full_name
  Writes to BOTH;       • Keep backfilling in-       constraints              • Drop legacy triggers
  Reads from old          flight delta writes      • Run async VALIDATE
```

---

## Phase 1: Expand

### Step 1.1: Non-Blocking DDL Migration (Release `v2.4.0`)
Add the new columns as **nullable** so existing application versions can continue inserting rows without knowing about the new columns.

```sql
-- migration: 20250501_expand_user_names.sql
SET lock_timeout = '2s';

-- Instant metadata-only operation in PostgreSQL (sub-millisecond lock)
ALTER TABLE users 
ADD COLUMN first_name VARCHAR(100),
ADD COLUMN last_name VARCHAR(100);
```

### Step 1.2: Deploy Application Version 1 (Dual-Write)
Deploy App `v2.4.0`:
* **Read Path**: Still queries `full_name`.
* **Write Path (Dual-Write)**: Splitting inputs in application logic:
  ```python
  # Application logic in UserRegistrationService
  def register_user(full_name_input):
      first, last = parse_name(full_name_input)
      db.execute("""
          INSERT INTO users (id, full_name, first_name, last_name, created_at)
          VALUES (:id, :full_name, :first, :last, NOW())
      """, id=gen_uuid7(), full_name=full_name_input, first=first, last=last)
  ```
* **Result**: All new writes now populate both legacy and new columns. Legacy rows still have `NULL` in the new columns.

---

## Phase 2: Migrate (Chunked Backfill & Validation)

### Step 2.1: Asynchronous Keyset Backfill Script
Run a background worker process that iterates over historical rows in bounded batches, yielding CPU and disk I/O:

```python
# scripts/backfill_user_names.py
import time
import psycopg2

BATCH_SIZE = 5000
last_id = "00000000-0000-0000-0000-000000000000"

conn = psycopg2.connect("postgresql://...")
conn.autocommit = True

while True:
    with conn.cursor() as cur:
        # Keyset pagination (id > last_id) prevents expensive OFFSET scans
        cur.execute("""
            SELECT id, full_name 
            FROM users 
            WHERE id > %s AND (first_name IS NULL OR last_name IS NULL)
            ORDER BY id ASC 
            LIMIT %s
        """, (last_id, BATCH_SIZE))
        
        rows = cur.fetchall()
        if not rows:
            print("Backfill complete! 100% of historical rows migrated.")
            break
            
        for user_id, full_name in rows:
            first, last = parse_name(full_name)
            cur.execute("""
                UPDATE users 
                SET first_name = %s, last_name = %s 
                WHERE id = %s AND (first_name IS NULL OR last_name IS NULL)
            """, (first, last, user_id))
            last_id = user_id

        print(f"Migrated batch up to ID: {last_id}")
        time.sleep(0.05) # Yield to avoid saturating replica WAL streaming
```

### Step 2.2: Add Dual-Horizon Constraints
Once the backfill reaches $100\%$, enforce constraints for all new writes without scanning historical data:

```sql
-- Step 2.2A: Enforce on NEW writes immediately ($O(1)$ lock)
ALTER TABLE users 
ADD CONSTRAINT check_users_first_name_not_null 
CHECK (first_name IS NOT NULL) NOT VALID;

ALTER TABLE users 
ADD CONSTRAINT check_users_last_name_not_null 
CHECK (last_name IS NOT NULL) NOT VALID;

-- Step 2.2B: Validate historical rows in background without blocking writers
ALTER TABLE users VALIDATE CONSTRAINT check_users_first_name_not_null;
ALTER TABLE users VALIDATE CONSTRAINT check_users_last_name_not_null;
```

### Step 2.3: Deploy Application Version 2 (Read from New)
Deploy App `v2.5.0`:
* **Read Path**: Now reads directly from `first_name` and `last_name`.
* **Write Path**: Continues writing to both `full_name` and new columns (insurance policy).
* **Observation**: Monitor for 48 hours. Ensure zero queries depend on `full_name`.

---

## Phase 3: Contract

### Step 3.1: Deploy Application Version 3 (Prune Legacy Dependency)
Deploy App `v2.6.0`:
* All application references to `full_name` have been completely removed.
* Application writes and reads **strictly** from `first_name` and `last_name`.

### Step 3.2: Drop Legacy Artifacts
Execute the final contraction migration safely:

```sql
-- migration: 20250520_contract_user_names.sql
SET lock_timeout = '2s';

-- Instant metadata-only drop
ALTER TABLE users DROP COLUMN full_name;
```

### Verification Checklist
- [x] Zero application downtime recorded during all deployments.
- [x] Zero lock timeout exceptions triggered in production logs.
- [x] 100% of 15,000,000 rows verified for first/last name presence.
- [x] Clean git diff: zero dead code referencing `full_name`.
