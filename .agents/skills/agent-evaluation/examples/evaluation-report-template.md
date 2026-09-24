# EVALUATION_REPORT Template

```markdown
# Agent Evaluation Report: [Task or Milestone Name]

> **Gate Verdict**: [CERTIFIED_READY | BLOCKED]
> **Timestamp**: [ISO 8601 Timestamp]
> **Working Directory**: [Workspace Path]

---

## 1. The 5-Point Verification Audit

| Gate | Check Name | Command / Inspection Method | Result | Evidence Citation |
| :---: | :--- | :--- | :---: | :--- |
| **G1** | **Requirement Traceability** | Line-by-line prompt checklist audit. | ✅ PASS | All 4 acceptance criteria implemented |
| **G2** | **Static Types & Linter** | `npx tsc --noEmit` | ✅ PASS | Exit code 0, 0 errors |
| **G3** | **Test Suite Execution** | `npm test` | ✅ PASS | 24 tests passed, 0 failed (1.8s) |
| **G4** | **Diff Sanity & Hygiene** | `git diff --stat` | ✅ PASS | 3 files modified (+45 -12), no debug prints |
| **G5** | **Negative Validation** | Executed hostile path test for invalid token. | ✅ PASS | Returned 401 with expected error schema |

---

## 2. Key Code Citations

- **Core Implementation**: `[CODE: src/auth/token-service.ts#L45-L78]`
- **Defensive Boundary**: `[CODE: src/auth/schemas.ts#L12-L28]`
- **Verification Tests**: `[CODE: tests/auth/token-service.test.ts#L15-L65]`

---

## 3. Residual Debt & Next Actions

- **Technical Debt Introduced**: None.
- **Recommended Follow-up**: Add rate-limiting middleware in Phase 2 deployment.
```
