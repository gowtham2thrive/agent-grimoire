# Specialized Review Lenses

> **Mandate**: When reviewing code, cognitive saturation leads to missed bugs. Audit the change surface by cycling systematically through four specialized review lenses, or delegating them to independent review subagents.

---

## <a id="lens-1"></a>Lens 1: Contract & Correctness

Focus exclusively on logic bugs, condition boundaries, and data integrity:

### Inspection Checklist:
1. **Condition & Branch Inversions**:
   - Are `if` statements inverted? (e.g. `!isValid` when `isValid` was intended).
   - Are ternary operators returning values in the reversed order?
2. **Boundary & Off-By-One Errors**:
   - Are slice/array indices bounded? Look closely at `<=` vs `<` and `>=` vs `>`.
   - What happens on empty collections, single-item collections, or maximum limit buffers?
3. **Null, Undefined & Optional Handling**:
   - Are optional fields dereferenced without optional chaining or null guards?
   - Can a database query return `null` and cause an unhandled runtime exception downstream?
4. **State Machine Integrity**:
   - Can an entity transition into an invalid or illegal state?
   - Are database mutations wrapped in transactions where partial failure leaves data inconsistent?

---

## <a id="lens-2"></a>Lens 2: Security & Vulnerabilities

Focus on untrusted inputs, authorization boundaries, and data exposure:

### Inspection Checklist:
1. **Injection Vectors (OWASP A03)**:
   - Are SQL queries concatenated using raw strings instead of parameterized queries?
   - Are shell commands executed with unescaped user inputs (`child_process.exec`, `os.system`)?
   - Is raw HTML or user markdown rendered without sanitization (XSS)?
2. **Authentication & Authorization Bypass (OWASP A01)**:
   - Does an endpoint verify that the authenticated user owns the resource being accessed or modified (IDOR prevention)?
   - Can an unauthenticated caller invoke admin or internal endpoints?
3. **Secret Hygiene**:
   - Are API keys, private tokens, passwords, or internal connection strings committed in code, configs, or test fixtures?
4. **SSRF & Path Traversal**:
   - Can user input dictate file paths (`../../etc/passwd`) or external webhook URLs to internal VPC networks?

---

## <a id="lens-3"></a>Lens 3: Performance & Resource Safety

Focus on memory retention, latency degradation, and resource exhaustion:

### Inspection Checklist:
1. **Unbounded Queries ($N+1$)**:
   - Is an ORM or database query executed inside a loop over collection items?
   - Are database queries missing pagination (`LIMIT` / `OFFSET`) on growing tables?
2. **Memory Leaks & Event Listeners**:
   - Are event listeners or subscriptions registered without corresponding removal on teardown/unmount?
   - Are cache maps growing unbounded without eviction policies (LRU/TTL)?
3. **Resource Handle Exhaustion**:
   - Are file descriptors, database client connections, or HTTP response bodies left unclosed?
4. **Event Loop & Thread Blocking**:
   - Is CPU-intensive computation (e.g. large file hashing, synchronous cryptographic operations) executed directly on the main event loop?

---

## <a id="lens-4"></a>Lens 4: Regression & Test Completeness

Focus on backward compatibility, caller impact, and verification coverage:

### Inspection Checklist:
1. **Public Contract Compatibility**:
   - Were existing function parameters changed, removed, or reordered without backward compatibility?
   - Does a modified API endpoint change the response schema in a way that breaks existing frontend or mobile clients?
2. **Untested Critical Branches**:
   - Did the pull request add new error handling, rollback, or boundary logic without corresponding unit or integration tests?
3. **Test Flakiness & Mock Drift**:
   - Were new tests written with hardcoded `sleep()` statements or weak assertions?
   - Are mocks assuming return signatures that differ from real production services?
