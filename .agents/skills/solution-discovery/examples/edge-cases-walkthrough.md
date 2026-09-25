# Solution Discovery: Edge-Case Scenarios & Resolution Traces

> **Context**: Autonomous agents encounter non-standard boundary conditions in enterprise and secure codebases. This document provides concrete execution traces for three difficult edge cases: a strict Zero-Dependency mandate, a Zombie library with 25k stars, and a Transitive License Infiltration attempt.

---

## Scenario 1 · The Zero-Dependency / Air-Gapped Mandate

### Context
A Rust-based cryptographic signing tool runs in an isolated, air-gapped hardware security module (HSM). The user prompt requests: *"Add RFC 7515 JSON Web Signature (JWS) compact serialization support."* The repository maintains a strict **Zero-New-Dependencies** rule for security certification.

### Execution Trace
1. **Constraint Ingress**:
   - Security Invariant: No external crates may be added to `Cargo.toml`.
   - External network access is disabled.
2. **Solution Spectrum Traversal**:
   - *Tier 1 (Standard Library)*: Rust `std` does not provide base64url or HMAC algorithms natively.
   - *Tier 2 (Framework)*: No web framework present.
   - *Tier 3 (Internal Composition)*: The project already imports `ring` (cryptographic primitives) and `serde_json` (JSON serialization).
3. **Synthesis via Internal Composition (Tier 3)**:
   - Instead of adopting an external `jsonwebtoken` crate, the agent composes the capability using existing project dependencies:
     - `ring::hmac` for HMAC-SHA256 signature computation.
     - `serde_json::to_vec` for payload serialization.
     - 25 lines of custom base64url encoding authored internally (Triviality Axiom).
4. **Outcome**:
   - Zero new dependencies introduced.
   - Fully satisfies RFC 7515 requirements.
   - Preserves air-gapped compliance and passes security audit.

---

## Scenario 2 · The Zombie Library Trap (25k Stars, Abandoned Code)

### Context
A Python backend needs a robust Slugify utility to convert arbitrary Unicode article titles into URL-safe ASCII slugs (handling Cyrillic, Greek, German umlauts, and Chinese pinyin).

### Execution Trace
1. **Candidate Discovery**:
   - The agent's parametric memory suggests `unicode-slugger` (25,000 GitHub stars, 1.2M monthly downloads).
2. **Epistemic Grounding Pass (Tier 1 Code Inspection)**:
   - Upstream Git repo inspection reveals:
     - Last commit: 4.5 years ago.
     - Open issues: 240, including *"Python 3.12 compatibility broken due to deprecated `cgi` module removal"*.
     - Open PRs: 48 pull requests un-reviewed by maintainer.
3. **Evaluation Gate Verdict**:
   - **Hard Veto Triggered**: *Axis 3 (Vitality & Maintenance)*. Vitality velocity score $V_v = 0.05$. Candidate is **Zombie Abandonware**.
4. **Spectrum Pivot**:
   - The agent inspects `awesome-slugify` vs. `python-slugify` vs. Python Standard Library.
   - Discovers `python-slugify` is actively maintained (last release 2 weeks ago), fully supports Python 3.12, 100% test pass rate, and uses unidecode/text-unidecode.
5. **Outcome**:
   - Avoids introducing a fatal runtime crash on Python 3.12+.
   - Selects the active, healthy library wrapped in a simple project adapter.

---

## Scenario 3 · The Transitive License Infiltration (Viral Copyleft)

### Context
A commercial SaaS application distributed under a proprietary license needs an XML digital signature verification tool.

### Execution Trace
1. **Candidate Discovery**:
   - Candidate `fast-xml-sig` is evaluated. The root `package.json` declares `"license": "MIT"`.
2. **Transitive Dependency Tree Audit (via `dependency-management`)**:
   - The agent inspects the resolved dependency graph:
     - `fast-xml-sig@1.2.0` (MIT)
       - `lib-xml-canon@0.8.0` (MIT)
         - `gpl-xpath-parser@2.1.0` (GPL-3.0-or-later) ⚠️
3. **Evaluation Gate Verdict**:
   - **Hard Veto Triggered**: *Axis 5 (Licensing Integrity)*.
   - The transitive dependency `gpl-xpath-parser` is licensed under GNU GPL v3.0, which imposes copyleft distribution obligations on derivative works. Incorporating this candidate into a proprietary product creates severe legal non-compliance.
4. **Alternative Selection**:
   - The agent rejects `fast-xml-sig` and selects `xml-crypto`, whose entire transitive dependency tree is verified to be 100% MIT and Apache-2.0 compliant.
5. **Outcome**:
   - Eliminates commercial legal liability before a single line of code is committed.
   - Generates a SAR explicitly noting the GPL rejection and the alternative choice.
