# The 5-Point Verification Checklist

> **Mandate**: Every requirement must be traceable to concrete code; every code change must be verified by passing compilation and tests; and every diff must be pristine.

---

## Gate 1: Requirement Traceability Matrix
- [ ] Every functional requirement stated in the user prompt maps to a specific function or module.
- [ ] All specified edge cases (e.g. empty lists, offline status, duplicate IDs) are handled.
- [ ] Any architectural constraints (e.g. "do not add new dependencies", "preserve backward compatibility") were honored.
- [ ] No speculative or unrequested features were added outside the user's objective.

---

## Gate 2: Static Analysis & Type Checking
- [ ] Project compiler or type-checker executed:
  - TypeScript: `npx tsc --noEmit`
  - Python: `mypy .` or `pyright`
  - Go: `go vet ./...`
  - Rust: `cargo check`
- [ ] Exit code is `0`.
- [ ] Zero unsuppressed type errors, unresolved imports, or broken type definitions.

---

## Gate 3: Test Suite Verification
- [ ] Scoped unit tests for the modified files executed and passed.
- [ ] Full regression test suite executed across the repository.
- [ ] Exit code is `0`.
- [ ] Real terminal output inspected to confirm test count matched expectations (i.e. tests were actually executed, not skipped).

---

## Gate 4: Diff Sanity & Hygiene
- [ ] Run `git diff HEAD` (or against target branch).
- [ ] No temporary files, debug scripts, or scratch files left in working tree.
- [ ] No leftover debug statements (`console.log`, `print`, `debugger`, `fmt.Println`).
- [ ] No formatting churn on files that had no functional edits.
- [ ] No private keys, passwords, or sensitive local tokens committed.

---

## Gate 5: Negative Boundary Validation
- [ ] Verified that passing malformed inputs triggers a clean, handled error rather than an unhandled crash or 500.
- [ ] Verified that error messages contain actionable domain context.
- [ ] Verified that resources are cleaned up on failure (no leaked connections or file descriptors).
