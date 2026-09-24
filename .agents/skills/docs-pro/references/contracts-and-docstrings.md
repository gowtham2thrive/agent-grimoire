# API Contracts & Code Documentation Guide

Code-level documentation must explain **why** code exists, what invariants it maintains, and its failure contracts — never restating self-evident syntax.

---

## 1. The Core Rule: Invariants Over Syntax

- **Self-evident (never write this)**: `// Sets the timeout` above `this.timeout = timeout;`
- **Valuable (always write this)**: `// Timeout covers TLS handshake + header parsing; connection drops hard when exceeded.`

Document **behavioral invariants**, **ownership semantics**, **thread-safety guarantees**, **side effects**, and **failure contracts**. Skip anything a competent reader can infer from the type signature alone.

---

## 2. API Contract Checklist

When documenting public interfaces — REST/GraphQL endpoints, SDK methods, CLI commands, tool declarations, or library exports — systematically capture:

1. **Parameters & Inputs**: Exact types, nullability, defaults, valid ranges, and format constraints.
2. **Return Value & Shape**: Success model, pagination cursors, and nullable/optional fields.
3. **Error & Failure Contracts**: What errors can occur, under what exact conditions, and how callers should recover.
4. **Side Effects**: Database writes, network calls, file mutations, lock acquisitions, event emissions.
5. **Concurrency**: Is it safe for concurrent use? What synchronization mechanism protects shared state?
6. **Idempotency**: Can the operation be safely retried? What key or mechanism prevents duplicate effects?

---

## 3. Language-Specific Patterns

The following are **representative examples** showing how to apply the contract checklist in different ecosystems. Adapt to whichever language and docstring convention the repository actually uses — these are not an exhaustive or closed set:

### Python (Google Style / PEP 257)
```python
def debit_account(account_id: str, amount: Decimal) -> TransactionRecord:
    """Debits funds from an active account within an isolated transaction.

    Verifies account standing and sufficient balance before applying debit.
    Emits an audit event upon successful ledger write.

    Args:
        account_id: Primary UUID of the debtor account.
        amount: Non-negative amount to deduct from balance.

    Returns:
        Immutable TransactionRecord with ledger timestamp and confirmation UUID.

    Raises:
        InsufficientFundsError: When balance is less than requested amount.
        AccountSuspendedError: When the account is locked against mutations.
    """
```

### TypeScript / JavaScript (TSDoc / JSDoc)
```typescript
/**
 * Resolves a DNS resource record with retries and exponential backoff.
 *
 * @param hostname - Fully qualified domain name to resolve.
 * @param recordType - DNS record type to query.
 * @param options - Timeout and custom nameserver overrides.
 * @returns Resolves with array of matched IP strings, or empty array if NXDOMAIN.
 * @throws {DnsTimeoutError} When all configured nameservers time out.
 */
export async function resolveDns(
  hostname: string,
  recordType: RecordType,
  options?: DnsQueryOptions
): Promise<string[]> {
```

### Go (GoDoc)
```go
// Allow reports whether an event with the given key may happen now.
// It is safe for concurrent use across multiple goroutines and uses
// atomic operations rather than mutex locks for low overhead.
func (b *Bucket) Allow(key string) bool {
```

### Rust (Rustdoc)
```rust
/// Reads an exact number of bytes from the socket into the provided buffer.
///
/// Blocks until the buffer is completely filled or the peer closes the
/// connection. Never leaves the buffer partially modified on error.
///
/// # Errors
///
/// Returns [`std::io::ErrorKind::UnexpectedEof`] if the stream reaches
/// EOF before the buffer is fully populated.
pub fn read_exact(&mut self, buf: &mut [u8]) -> std::io::Result<()> {
```

### Other Ecosystems

For languages not shown above (Java/Javadoc, C#/XML doc comments, Swift/DocC, Kotlin/KDoc, C++/Doxygen, Ruby/YARD, etc.), apply the same contract checklist — parameters, returns, errors, side effects, concurrency — using the ecosystem's standard doc comment format. Inspect existing docstrings in the repository to match prevailing style.
