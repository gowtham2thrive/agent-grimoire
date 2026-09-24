# Severity Scoring & Evidence Certification: High-Signal Auditing Protocol

> **Mandate**: The greatest enemy of effective security engineering is noise. False positives and vague, speculative warnings destroy developer trust and paralyze agent workflows. Every candidate finding must be scored using an objective 0–100 confidence rubric based on concrete code evidence. Any finding scoring below 70 is strictly suppressed.

---

## 1 · The 0–100 Confidence Rubric

Before recording a security finding in an audit report, PR review, or terminal output, calculate its confidence score using this formula:

$$\text{Confidence Score} = \mathbf{P}_{\text{reach}} + \mathbf{P}_{\text{exploit}} + \mathbf{P}_{\text{blast}} + \mathbf{P}_{\text{patch}} - \mathbf{P}_{\text{speculation}}$$

| Component | Points | Criteria & Requirements |
| :--- | :--- | :--- |
| **$\mathbf{P}_{\text{reach}}$: Reachability Proof** | **40 pts** | Full source-to-sink path proven with line citations (or public API export proof for libraries). If reachability is broken or dead code, award 0 pts. |
| **$\mathbf{P}_{\text{exploit}}$: Reproducible Exploit** | **30 pts** | A concrete, realistic hostile input payload and execution sequence that triggers the failure mode. |
| **$\mathbf{P}_{\text{blast}}$: Blast Radius Clarity** | **20 pts** | Clear explanation of the direct impact (e.g. data breach, arbitrary code execution, privilege escalation, persistent state corruption). |
| **$\mathbf{P}_{\text{patch}}$: Deterministic Remediation** | **10 pts** | A minimal, verified code diff or configuration change that eliminates the vulnerability without breaking functionality. |
| **$\mathbf{P}_{\text{speculation}}$: Speculation Penalty** | **-30 pts** | Deducted if the finding relies on ungrounded assumptions ("If an admin alters the database schema in the future..."). |

---

## 2 · The Strict Suppression Rule

```
[Candidate Security Finding Formulated]
                 │
                 ▼
     [Calculate Confidence Score]
                 │
        ┌────────┴────────┐
        ▼                 ▼
   Score < 70        Score ≥ 70
        │                 │
        ▼                 ▼
 [AUTO-DISCARD]    [CLASSIFY SEVERITY (P0..P3)]
 (Zero output in    (Include in auditable report
  final report)      with patch diff)
```

> [!WARNING]
> **Zero Cosmetic Bikeshedding**:
> 1. Any candidate finding with a confidence score **below 70** is **strictly suppressed**. Agents are forbidden from reporting hypothetical or unprovable concerns.
> 2. Feedback on code formatting, whitespace, indentation, or subjective variable naming is **strictly forbidden**. Those belong to automated formatting tools.

---

## 3 · Severity Tiers & SLA

For findings that pass the 70-point threshold, assign severity based on impact and reachability:

* **P0 - Blocker (Score $\ge 90$)**:
  * *Impact*: Active Remote Code Execution (RCE), unauthenticated data exfiltration, root privilege escalation, agent tool escape, or plaintext high-entropy production credentials committed.
  * *Action*: Immediate merge blocker; halts CI/CD pipeline.
* **P1 - Critical (Score $\ge 80$)**:
  * *Impact*: Authenticated IDOR/BOLA, unparameterized database query on internal API, memory corruption under reachable inputs, prompt injection hijacking state-mutating agent tools.
  * *Action*: Must be remediated before production release.
* **P2 - Moderate (Score $\ge 70$)**:
  * *Impact*: Missing perimeter validation on secondary endpoints, weak cryptographic cipher mode, reachable dependency with known low-to-medium CVE, missing rate limits.
  * *Action*: Tracked as prioritized technical debt; remediated in current milestone.
* **P3 - Advisory (Score $\ge 70$)**:
  * *Impact*: Defense-in-depth architectural improvement, missing audit log for non-destructive actions, unpinned dependency with zero current reachability.
  * *Action*: Non-blocking recommendation.

---

## 4 · Structured Security Report Schema

When outputting security findings for code reviews, architecture audits, or certification gates, adhere strictly to this format:

```markdown
### [SEV-{P0..P3}] {Concise Vulnerability Title}

- **Invariant Violated**: Invariant {1..7} ({Invariant Name})
- **Confidence Score**: {Score}/100 (Reachability: {pts}, Exploit: {pts}, Blast: {pts}, Patch: {pts})
- **Taint / Reachability Trace**:
  - `Source`: [`path/to/file.ts#L42`](file:///path/to/file.ts#L42) (`req.body.id`)
  - `Intermediate`: [`path/to/service.ts#L88`](file:///path/to/service.ts#L88) (`id` passed unvalidated)
  - `Sink`: [`path/to/db.ts#L105`](file:///path/to/db.ts#L105) (string-concatenated SQL query)
- **Concrete Exploit Scenario**:
  An attacker sends an HTTP POST request with payload `{"id": "1; DROP TABLE users;--"}`. Because the identifier is interpolated directly into the database query without parameterization, the query terminates early and executes the drop command.
- **Blast Radius**:
  Complete database compromise and data loss across all customer tenants.
- **Verified Remediation Diff**:
  ```diff
  - const query = `SELECT * FROM tenants WHERE id = '${id}'`;
  - return await db.query(query);
  + const query = `SELECT * FROM tenants WHERE id = $1`;
  + return await db.query(query, [id]);
  ```
```
