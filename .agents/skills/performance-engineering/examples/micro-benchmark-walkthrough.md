# Micro-Benchmark Walkthrough: The 3-Line Intent Protocol

> **Purpose**: Demonstrates how an autonomous agent applies the lightweight `micro-benchmark` mode to quickly optimize a local routine without heavy enterprise paperwork or cognitive delay.

---

## 1 · Scenario Context

An agent is modifying a telemetry export utility where an $O(N^2)$ array filtering operation is executed inside a tight batch loop processing 5,000 log records.

---

## 2 · Step-by-Step Execution Trace

### Step 1: Pre-Mutation Benchmark (Zero Ceremony)
The agent runs a localized micro-benchmark or timing script:
```bash
# Example quick timing run
node benchmark_export.js
# Output: batch_filter_records: 18.42ms (mean of 50 runs), memory delta: 3.2MB
```

### Step 2: Emit the 3-Line Performance Intent Block
Directly preceding the code mutation in the response or execution log, the agent outputs the exact 3-line block:

```markdown
> **Baseline Metric**: 18.42ms per 5k batch / 3.2MB allocation / O(N^2) array lookup
> **Bottleneck & Mutation**: Replace nested array `.includes()` scan with pre-indexed `Set` lookup (O(N) total)
> **Verified Post-Metric**: 0.84ms per batch (-95.4% latency) / 0.4MB allocation / 100% tests green
```

### Step 3: Apply the Minimal Mutation
```javascript
// BEFORE (O(N^2)):
export function filterActiveEvents(events, excludedIds) {
  return events.filter(e => !excludedIds.includes(e.id));
}

// AFTER (O(N) with Set):
export function filterActiveEvents(events, excludedIds) {
  const excludedSet = new Set(excludedIds);
  return events.filter(e => !excludedSet.has(e.id));
}
```

### Step 4: Verification Command
```bash
npm test && node benchmark_export.js
```
The test passes, the benchmark confirms execution time dropped from $18.42\text{ms}$ to $0.84\text{ms}$, and the task is immediately complete.
