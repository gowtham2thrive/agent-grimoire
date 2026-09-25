# Micro-Mode Planning Walkthroughs

> **Mandate**: For changes $< 30$ lines localized to a single file or function, never generate markdown files, complex DAGs, or bureaucratic paperwork. Emit a concise **3-Line Execution Intent** directly before mutating code, run the verification command, and verify completion.

---

## Example 1 · Rust Systems Utility (Boundary Fix)

### Scenario
An off-by-one buffer index panic in a token parser (`src/parser/lexer.rs`).

### The 3-Line Execution Intent Block
```markdown
> **Target**: `src/parser/lexer.rs::tokenize_ident` (Line 142)
> **Mutation**: Change `cursor + len <= buffer.len()` to prevent out-of-bounds slice indexing on EOF.
> **Verification**: `cargo test --lib parser::lexer::tests::test_eof_ident_handling`
```

### Execution & Verification
1. Run pre-check: `cargo test --lib parser::lexer::tests::test_eof_ident_handling` (fails with index out of bounds).
2. Apply mutation in `src/parser/lexer.rs`.
3. Run post-verification: `cargo test --lib parser::lexer::tests::test_eof_ident_handling` (passes green, exit code 0).
4. Run regression check: `cargo check` (passes green).

---

## Example 2 · TypeScript API Handler (Null Pointer Defense)

### Scenario
Uncaught `TypeError: Cannot read properties of undefined (reading 'tier')` when a user organization has null billing metadata.

### The 3-Line Execution Intent Block
```markdown
> **Target**: `src/api/middleware/subscription.ts::checkTierAccess` (Line 28)
> **Mutation**: Add optional chaining and fallback: `req.org?.billing?.tier ?? 'free'`.
> **Verification**: `npx vitest run src/api/middleware/subscription.test.ts`
```

### Execution & Verification
1. Run pre-check: `npx vitest run src/api/middleware/subscription.test.ts` (1 failing test reproducing null billing).
2. Apply mutation in `src/api/middleware/subscription.ts`.
3. Run post-verification: `npx vitest run src/api/middleware/subscription.test.ts` (all 6 tests pass green).

---

## Example 3 · Python Data Pipeline (Pagination Cursor Edge Case)

### Scenario
A data pipeline consumer in `pipelines/ingest.py` drops the last record when `has_more` is false but `records` is non-empty.

### The 3-Line Execution Intent Block
```markdown
> **Target**: `pipelines/ingest.py::paginate_stream` (Line 73)
> **Mutation**: Yield records before evaluating `cursor = resp.get('next_cursor')` termination condition.
> **Verification**: `pytest tests/test_ingest.py -k test_paginate_stream_terminal_batch -v`
```

### Execution & Verification
1. Run pre-check: `pytest tests/test_ingest.py -k test_paginate_stream_terminal_batch` (assert 100 == 99 fails).
2. Apply mutation in `pipelines/ingest.py`.
3. Run post-verification: `pytest tests/test_ingest.py -k test_paginate_stream_terminal_batch` (passes green).
