# Secrets Hygiene & Supply Chain Verification

> **Mandate**: Secrets in code and unpinned external dependencies are two of the most common vectors of systemic compromise. Enforce deterministic credential lifecycles using Shannon entropy combined with semantic context binding. Audit third-party supply chains through reachability-oriented analysis and cryptographic digest pinning.

---

## 1 · The Triad Rule for Secret Detection

Never rely on brittle, vendor-specific regular expressions alone. To avoid false-positive hallucinations on benign hashes (Git commit SHAs, SHA-256 checksums, UUIDs, Base64 assets), a valid secret finding **must satisfy all three conditions of the Secret Triad**:

```mermaid
flowchart TD
    E["1. High Shannon Entropy<br/>(Random character distribution H(X) > 4.5)"]
    S["2. Semantic Intent Binding<br/>(Assigned to credential-bearing identifier)"]
    X["3. Non-Exclusion Test<br/>(NOT pure hex hash, UUID, SVG, or test fixture)"]

    E & S & X -->|All 3 Satisfied| ALERT["CONFIRMED LEAKED SECRET (P0/P1)"]
    E & S -->|Fails Non-Exclusion| PASS["BENIGN HASH / ASSET (Discard)"]
```

### 1.1 Condition 1: High Shannon Entropy
Cryptographic keys and private tokens exhibit high information entropy:
$$H(X) = -\sum_{i=1}^n P(x_i) \log_2 P(x_i)$$
* Typical natural language or code keywords: $H(X) \approx 2.5 - 3.5$.
* Base64 / Hex cryptographic tokens: $H(X) \ge 4.5$ bits per character.

### 1.2 Condition 2: Semantic Intent Binding
The high-entropy string must be bound to an identifier, environment variable, or configuration key indicating credential storage:
* Identifiers: `apiKey`, `api_token`, `secret`, `private_key`, `client_secret`, `access_token`, `password`, `auth_header`, `bearer`.
* Connections: `postgres://user:pass@host`, `mongodb+srv://...` with embedded credentials.

### 1.3 Condition 3: Format Exclusions (False-Positive Elimination)
Immediately discard candidates that match any of the following non-secret formats:
- **Pure Hex Hashes**: 32-character (MD5), 40-character (SHA-1), or 64-character (SHA-256) hexadecimal strings without credential semantic context (e.g. Git commit IDs, integrity checksums, cache keys).
- **UUIDs**: Standard 8-4-4-4-12 hex format (e.g. `123e4567-e89b-12d3-a456-426614174000`).
- **Media Assets**: Base64 data URIs starting with `data:image/`, SVG vector paths, or font byte dumps.
- **Isolated Test Fixtures**: Obvious mock strings located strictly within test directories (e.g. `dummy_token_12345`, `mock_jwt_secret`, `test-api-key`).

---

## 2 · The Secret Lifecycle Invariants

1. **Zero Persistence in Source**: Secrets must never be committed to source code repositories, commit history, configuration files, build logs, or client-side assets.
2. **Ephemeral Runtime Injection**: Credentials must be injected into the running process at startup via secure environment variables, memory-mapped tmpfs files, or dynamic secret managers (AWS Secrets Manager, GCP Secret Manager, Vault).
3. **Capability & Time Scoping**: Never issue broad, permanent root API keys. Prefer short-lived, cryptographically scoped session tokens (e.g. OAuth2 tokens, AWS STS temporary credentials, GitHub Actions OIDC tokens) that expire within minutes or hours.
4. **Git Hygiene & History Scrubbing**: If a secret is committed accidentally:
   - Immediately rotate/revoke the credential upstream.
   - Removing the secret in a follow-up commit is insufficient; scrub the git tree (e.g. via git filter-repo or BFG) or assume the credential has been archived by public scrapers.

---

## 3 · Zero-Trust Supply Chain & Dependency Verification

Third-party dependencies are external code running inside your trust perimeter. Audit them with the same reachability and verification discipline applied to first-party code:

### 3.1 Cryptographic Digest Pinning
* **Floating Ranges are Dangerous**: Ranges like `^1.2.3` or `~2.0` permit automated upstream pull of compromised or hijacked minor/patch versions.
* **Deterministic Lockfiles**: Every project must enforce exact version locking with cryptographic hash verification:
  - Node.js: `package-lock.json` or `pnpm-lock.yaml` with `integrity: sha512-...`
  - Python: `poetry.lock` or `uv.lock` with recorded sha256 digests.
  - Rust: `Cargo.lock` with package checksums.
  - Go: `go.sum` with recorded cryptographic module hashes.

### 3.2 Reachability-Oriented SCA (Software Composition Analysis)
Do not panic over raw CVE counts. Apply **Reachability-Oriented Triage**:

```
[Vulnerable Library L discovered in lockfile]
       │
       ▼
[Is library L actually imported in source code?]
  ├── NO  ──► [Classify: Unused Transitive Dependency] (Low Priority Debt)
  └── YES ──► [Is vulnerable function F() invoked?]
                ├── NO  ──► [Classify: Unreachable Vulnerability] (P3 Advisory)
                └── YES ──► [Does untrusted input reach F()?]
                              ├── NO  ──► [Classify: Internal Reachable] (P2 Moderate)
                              └── YES ──► [ACTIVE EXPLOITABLE CVE] (P0/P1 Blocker)
```

1. **Presence $\ne$ Exploitability**: A known CVE in a library function that is never imported, invoked, or reachable from untrusted ingress is maintenance debt, not an active security emergency.
2. **Verify Call Graph**: Trace from internal application entry points to the imported third-party symbol. If no execution path reaches the vulnerable function, suppress the alert or classify it strictly as an informational update.
