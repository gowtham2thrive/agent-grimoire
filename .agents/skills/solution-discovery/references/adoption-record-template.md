# Solution Adoption Record (SAR) Template

> **Mandate**: Every material technology selection, library adoption, or custom-build architectural decision must be permanently recorded. Future maintainers and autonomous agents must be able to understand *why* a solution was selected, what alternatives were rejected, and what operational trade-offs were accepted.

---

## 1 · SAR Markdown Schema

Create records under `docs/adr/` or `docs/decisions/` using the naming convention `SAR-XXX-short-title.md` (e.g. `SAR-004-rate-limiting-engine.md`).

```markdown
# SAR-[000]: [Adopt / Compose / Build] [Capability / Library Name]

- **Status**: [Proposed | Accepted | Rejected | Superseded by SAR-YYY]
- **Date**: YYYY-MM-DD
- **Decision Owner**: [Agent Role / Engineer Name]
- **Solution Spectrum Tier**: [Tier 1: Built-in | Tier 2: Framework | Tier 3: Compose | Tier 4: Vendor | Tier 5: Adopt | Tier 6: Build]
- **Relevant Subsystems**: [e.g. `src/core/auth`, `src/infrastructure/cache`]

---

## 1. Context & Capability Requirement
- **Problem Statement**: What problem does this capability solve?
- **Operational Envelope**:
  - Expected throughput / QPS: [e.g. 5,000 QPS peak]
  - Latency budget: [e.g. < 5ms at p99]
  - Concurrency & Environment: [e.g. Node.js cluster, multi-tenant]
- **Success Invariants**: What non-negotiables must this solution satisfy?

---

## 2. Options Explored

### Option 1 (Selected): [Name & Version]
- **Category**: [External Package / Framework API / Custom Build]
- **Repository / Provenance**: [GitHub URL / Package Registry link]
- **Summary**: Brief description of how it works.

### Option 2 (Rejected): [Name & Version]
- **Category**: [External Package / Tool]
- **Reason for Rejection**: Why did this candidate fail? (Cite specific 10-axis failure, e.g. license, abandonware, bloat).

### Option 3 (Rejected / Considered): [Bespoke Custom Build]
- **Reason for Rejection / Adoption**: Evaluation of in-house build cost vs. adoption TCO.

---

## 3. The 10-Axis Evaluation Scorecard

| Axis | Option 1 (Selected) | Option 2 (Rejected) | Custom Build |
| :--- | :---: | :---: | :---: |
| **1. Technical Fit** | 9 / 10 | 8 / 10 | 8 / 10 |
| **2. Project Compatibility** | 10 / 10 | 6 / 10 (needs C-ext) | 10 / 10 |
| **3. Vitality & Maintenance** | 9 / 10 (commit last wk) | 2 / 10 (zombie 2 yrs) | 10 / 10 |
| **4. Security & Supply Chain** | 10 / 10 (0 CVEs) | 5 / 10 (unpatched High) | 10 / 10 |
| **5. Licensing Integrity** | PASS (MIT) | PASS (Apache-2.0) | PASS (Internal) |
| **6. Maturity & Quality** | 9 / 10 (98% cov) | 4 / 10 (no tests) | 8 / 10 |
| **7. Integration Cost & TCO** | 9 / 10 (0 deps) | 3 / 10 (45 deps) | 7 / 10 |
| **8. Performance & Resources** | 9 / 10 (< 1ms) | 7 / 10 | 10 / 10 |
| **9. Ecosystem & Docs** | 9 / 10 | 4 / 10 | N/A |
| **10. Capability Overlap** | 10 / 10 (no overlap) | 3 / 10 (duplicate) | 10 / 10 |
| **Composite Score** | **94 / 100** | **42 / 100** | **83 / 100** |

---

## 4. Decision & Trade-Off Rationale
- **Primary Driver**: Why this specific option was chosen.
- **Accepted Trade-Offs**: What limitations or downsides are we knowingly accepting? (e.g. slightly higher memory consumption, dependency upgrade tracking).
- **Anti-Corruption Isolation Plan**: What project interface encapsulates this choice? (File path to Port and Adapter).

---

## 5. Supply-Chain & Security Audit
- **License Verified**: [MIT / Apache-2.0 / BSD] (Confirmed no transitive copyleft contamination).
- **Vulnerability Scan**: 0 known CVEs in current release.
- **Build Hook Verification**: Confirmed package contains zero `preinstall` or arbitrary execution scripts.
- **Lockfile Hash Coherence**: Lockfile updated cleanly with pinned cryptographic checksums.

---

## 6. Verification Evidence
- **Adapter Test Suite**: `[PASS: tests/adapters/rate_limiter_adapter.spec.ts]`
- **Benchmark / Smoke Run**: [Output or execution log verifying latency/memory criteria]
- **Clean Install Verification**: `npm ci` / `cargo test` passes 100% in fresh environment.
```
