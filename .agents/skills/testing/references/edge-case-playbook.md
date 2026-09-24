# Testing Edge-Case Playbook

> **Mandate**: Real-world repositories have legacy debt, missing test harnesses, spaghetti dependencies, and non-deterministic logic. Execute these concrete fallback protocols to maintain rigorous verification without breaking down.

---

## 1. The Zero-Harness Brownfield Repository

### Problem
The project has 0 tests, or the test runner is unconfigured, broken, or requires proprietary corporate credentials not present in the environment.

### The Grimoire Protocol
1. **Detect**: In Phase 1 (Harness Discovery), confirm that no test runner configuration exists.
2. **Offer Scaffolding**: Check if the language has a zero-dependency native runner:
   - Node.js: `node --test` (native since Node 18+)
   - Python: `python -m unittest` (stdlib)
   - Go: `go test` (built-in)
   - Rust: `cargo test` (built-in)
3. **Fallback to Ephemeral Verification Scripts**:
   - If adding a permanent test framework is out of scope for the current prompt, write a self-contained validation script (e.g. `verify_patch.py` or `verify_fix.mjs`).
   - The script must exercise the modified function, assert inputs and outputs, and exit with code `0` on success or non-zero on failure.
   - Run the script: `node verify_fix.mjs`
   - Capture the output as verification evidence, and delete the temporary script before concluding.

---

## 2. Untestable Spaghetti & Legacy Code (Missing Seams)

### Problem
A monolithic 1,500-line legacy function needs a bug fix, but it accesses global state, reads static config files, and cannot be imported in a unit test without crashing.

### The Grimoire Protocol
1. **Never Perform Wide-Scale Speculative Refactoring**: Rewriting the entire legacy module to "make it testable" will introduce unforeseen regressions.
2. **Characterization / Golden Master Testing**:
   - Run the legacy function with existing inputs and capture the raw output (file, stdout, or response object).
   - Save this output as a "Golden Master" snapshot.
3. **Introduce a Surgical Seam**:
   - Apply Michael Feathers' *Working Effectively with Legacy Code* dependency injection seam:
   ```typescript
   // Legacy signature:
   function processMonthlyInvoices() { ... uses GlobalDatabase.connect() ... }

   // Surgical Seam:
   function processMonthlyInvoices(dbOverride?: DatabaseClient) {
     const db = dbOverride ?? GlobalDatabase.connect();
     ...
   }
   ```
   - Existing callers remain 100% unaffected (backward compatible), while your new test can inject a test stub.

---

## 3. Stochastic & Floating-Point Calculations

### Problem
Calculations involving floating-point numbers (`0.1 + 0.2 === 0.30000000000000004`), probabilistic algorithms, or machine learning models produce slight variances between environments.

### The Grimoire Protocol
1. **Epsilon / Delta Tolerances**: Never use strict equality (`toBe` / `==`) on floating-point numbers. Use epsilon tolerance checks:
   - TypeScript / Jest: `expect(actual).toBeCloseTo(expected, 5)` (within 5 decimal digits)
   - Python: `assert math.isclose(actual, expected, rel_tol=1e-5)`
   - Go: `if math.Abs(actual - expected) > 1e-6 { t.Fail() }`
2. **Seed Fixation**: In any test generating pseudo-random numbers or shuffling collections, explicitly set the random seed (`random.seed(42)`, `np.random.seed(42)`) at the start of the test.
3. **Invariant Bounds**: For stochastic simulations (e.g. Monte Carlo), assert statistical bounds (e.g. `mean >= 0.48 && mean <= 0.52`) with a sufficiently large sample size, rather than expecting an exact value.

---

## 4. Sandboxed / Offline Runtimes (No Docker / No Internet)

### Problem
Tests require a live PostgreSQL database, Redis instance, or third-party OAuth endpoint, but the agent execution sandbox has no internet connection or Docker daemon.

### The Grimoire Protocol
1. **Swap External Drivers with Embedded In-Memory Adapters**:
   - PostgreSQL $\rightarrow$ SQLite `:memory:` with compatible SQL dialects or PGLite.
   - Redis $\rightarrow$ `ioredis-mock` or in-memory key-value dictionary.
   - HTTP Services $\rightarrow$ Local mock server (`nock`, MSW, or native Node `http.createServer`).
2. **Boundary Contract Validation**: Verify that the adapter's input/output schema strictly matches the interface required by the core application.
