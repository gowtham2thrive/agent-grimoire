# Example: Full-Stack Feature Decomposition

A concrete walkthrough of decomposing a real feature ("User Profile Avatar Upload") into an orchestrated multi-agent plan. This example demonstrates a **phased** topology — shared foundation first, then parallel workers.

---

## The User's Request

> "Add avatar upload to user profiles. Users should be able to upload an image via the profile page, it gets stored in cloud storage, and the profile displays the avatar. Include proper validation (file type, size limits) and tests."

---

## Phase 0 — Triage & Topology Selection

### Independence Analysis

The task involves:
1. **Backend API**: New upload endpoint, storage integration, database schema change.
2. **Frontend UI**: New upload component, image preview, profile display.

These touch different directories (`src/api/` vs `src/components/`) but share:
- A database migration adding an `avatar_url` column to the `users` table.
- A TypeScript type definition for the updated `User` interface.
- A new dependency (`multer` or equivalent for file uploads).

### Topology Decision

**`phased`** — The shared schema migration, type update, and dependency installation must be committed before workers can proceed independently. After that, backend and frontend are genuinely parallel.

**Rejected alternatives**:
- `direct`: The task is large enough to benefit from parallel execution.
- `parallel`: Workers share a database schema and TypeScript type — divergence is guaranteed without foundation.
- `single_worker`: Two clearly independent domains (API vs UI) after foundation.

---

## Phase 1 — Foundation Settlement

The orchestrator implements and commits:

### 1.1 Database Migration
```sql
-- migrations/20260924_add_avatar_url.sql
ALTER TABLE users ADD COLUMN avatar_url TEXT DEFAULT NULL;
```

### 1.2 Updated TypeScript Type
```typescript
// src/types/user.ts
export interface User {
  id: string;
  email: string;
  name: string;
  avatarUrl: string | null;  // ← NEW
  createdAt: Date;
  updatedAt: Date;
}
```

### 1.3 Dependencies
```bash
npm install multer @types/multer
```

### 1.4 Foundation Commit
```bash
git add -A
git commit -m "feat(foundation): avatar upload schema, types, and deps"
```

**Verification**: `npx tsc --noEmit` passes. Existing tests pass. The foundation is clean.

---

## Phase 2 — Task Contract Formulation

### Worker A: `avatar-api`

```json
{
  "description": "Implement the avatar upload API endpoint with cloud storage integration and file validation",
  "read_first": ["src/types/user.ts", "src/api/routes.ts", "src/config/storage.ts"],
  "allowed_paths": ["src/api/avatar/**", "src/services/storage/**", "test/api/avatar/**"],
  "forbidden_paths": ["src/components/**", "src/types/user.ts", "package.json", "migrations/**"],
  "validation_command": ["npm", "test", "--", "test/api/avatar"],
  "definition_of_done": "POST /api/avatar accepts multipart file upload (JPEG/PNG, max 5MB), stores in cloud storage, updates user.avatarUrl in DB, returns { avatarUrl: string }. GET /api/avatar/:userId returns the URL. 8+ unit tests covering upload, validation errors, size limits, and invalid file types."
}
```

### Worker B: `avatar-ui`

```json
{
  "description": "Build the React avatar upload component with preview, drag-and-drop, and profile display integration",
  "read_first": ["src/types/user.ts", "src/components/Profile/ProfilePage.tsx"],
  "allowed_paths": ["src/components/Avatar/**", "src/components/Profile/ProfilePage.tsx", "test/components/Avatar/**"],
  "forbidden_paths": ["src/api/**", "src/services/**", "src/types/user.ts", "package.json"],
  "validation_command": ["npm", "test", "--", "test/components/Avatar"],
  "definition_of_done": "AvatarUpload component with drag-and-drop, file type/size client-side validation, image preview before upload, upload progress indicator, and integration into ProfilePage. 6+ component tests covering render, file selection, validation errors, and upload trigger."
}
```

### DECISIONS.md Entry

```markdown
### AD-1: Avatar Storage Strategy
- **Context**: Need to store user-uploaded avatar images.
- **Decision**: Use cloud storage (S3-compatible) with signed URLs for reads.
- **Consequences**: Backend owns upload; frontend uses returned URL for display.

### Shared Contract: User Type
- **Path**: `src/types/user.ts`
- **Owned by**: Foundation (committed, immutable during this orchestration)
- **Consumed by**: avatar-api, avatar-ui

### File Ownership
| Worker | Owns | Must Not Touch |
|--------|------|---------------|
| avatar-api | `src/api/avatar/**`, `src/services/storage/**`, `test/api/avatar/**` | `src/components/**`, `package.json` |
| avatar-ui | `src/components/Avatar/**`, `src/components/Profile/ProfilePage.tsx`, `test/components/Avatar/**` | `src/api/**`, `src/services/**`, `package.json` |
```

---

## Phase 3 — Dispatch

```bash
# Create isolated worktrees from foundation baseline
git worktree add -b agent/avatar-api .agents/worktrees/avatar-api main
git worktree add -b agent/avatar-ui .agents/worktrees/avatar-ui main

# Dispatch both workers concurrently (single batch)
# Each receives its rendered prompt with full context
```

Workers run in parallel. `avatar-api` builds the backend; `avatar-ui` builds the frontend. Neither touches the other's files.

---

## Phase 4 — Supervision

Both workers complete within 12 minutes. `avatar-ui` had one soft restart (forgot to import the `User` type from the correct path after reading the wrong file initially — corrected with targeted instructions).

---

## Phase 5 — Reconciliation & Verification

### Merge
```bash
git checkout main
git merge --no-ff agent/avatar-api -m "feat(api): avatar upload endpoint + storage integration"
git merge --no-ff agent/avatar-ui -m "feat(ui): avatar upload component + profile integration"
```

**Result**: Clean merge, no conflicts. Workers touched completely disjoint file sets.

### Integration Gate
```bash
npm test          # 247/247 passing (including 14 new avatar tests)
npx tsc --noEmit  # No type errors
npm run build     # Clean production bundle
npx eslint .      # No lint violations
```

### Synthesis Report

```markdown
## Orchestration Synthesis

### Objective
Add avatar upload to user profiles with storage, validation, and tests.

### Topology
phased — Shared schema migration and User type committed as foundation before parallel backend/frontend work.

### Foundation
Database migration (avatar_url column), updated User type, multer dependency.

### Worker Results
| Worker | Scope | Status | Key Deliverable | Restarts |
|--------|-------|--------|-----------------|----------|
| avatar-api | `src/api/avatar/**` | ✅ Complete | Upload endpoint + storage service + 8 tests | 0 |
| avatar-ui | `src/components/Avatar/**` | ✅ Complete | Upload component + profile integration + 6 tests | 1 (soft) |

### Integration Verification
- Tests: ✅ 247/247 passing
- Types: ✅ No errors
- Build: ✅ Clean
- Lint: ✅ No violations

### Conflicts Resolved
None — clean merge (disjoint file ownership).

### Notable Decisions
- avatar-ui soft-restarted once: worker initially imported User from wrong path.
  Corrective prompt pointed to `src/types/user.ts`. Resolved in second attempt.
```

---

## What Could Go Wrong (Anti-Pattern Illustrations)

### ❌ Skipping Foundation
If we had launched both workers without committing the `User` type update:
- `avatar-api` would have added `avatarUrl` to its local copy of the type.
- `avatar-ui` would have added `avatarUrl` to its local copy of the type.
- Merge: **conflict** on `src/types/user.ts` with two incompatible edits.

### ❌ Overlapping Path Boundaries
If `avatar-ui` were also allowed to modify `src/api/**`:
- Both workers might create competing upload endpoint implementations.
- Merge: **irreconcilable semantic conflict**.

### ❌ Trusting Individual Reports
If we had skipped the integration gate after both workers reported "all tests pass":
- A missing import or incompatible function signature between the API response format and the UI's expected data shape would only surface in production or during manual testing.
