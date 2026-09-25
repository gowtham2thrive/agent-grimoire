# AI & LLM Inference Optimization Walkthrough

> **Purpose**: Demonstrates performance engineering applied to an autonomous AI agent swarm and LLM inference gateway, optimizing TTFT, memory footprint, and token generation throughput.

---

## 1 · Scenario Context

An enterprise multi-agent workflow executes high-volume LLM inference against a 70B parameter open-weights model hosted on a cluster of $4\times \text{NVIDIA A100 (80GB)}$ GPUs.
- **Problem**: As agent turns accumulate, Time to First Token (TTFT) degrades to $> 3.2\text{ seconds}$, and concurrent agent requests fail with GPU Out-of-Memory (`CUDA OOM`) during long reasoning steps.
- **Requirement**: Reduce TTFT $< 600\text{ms}$ and double concurrent serving capacity without exceeding GPU VRAM.

---

## 2 · Step-by-Step Execution Trace

### Phase 1: Baseline Metrics Capture ($M_{\text{pre}}$)
- Workload: 64 concurrent agent streams, each with an 8k shared system instruction/grimoire context and 2k dynamic context.
- **Metrics**:
  - TTFT ($P_{99}$): $3,450\text{ms}$ (dominated by 8k prompt prefill on every turn)
  - Inter-Token Latency (ITL, $P_{99}$): $48\text{ms/token}$
  - Peak GPU VRAM per GPU: $77.8\text{GB}$ (near $80\text{GB}$ ceiling, crashing with OOM at 70 concurrent users)
  - Throughput: $310\text{ tokens/sec}$ total across cluster

---

### Phase 2: Profiling & Bottleneck Isolation
Using PyTorch Profiler and inference server metrics:
1. **Redundant Prefill Compute**: Every agent turn recomputed attention matrices over the identical 8,000-token system instruction, wasting $> 70\%$ of GPU Tensor Core FLOPs.
2. **KV-Cache Fragmentation**: Naive contiguous memory allocation wasted $58\%$ of VRAM due to worst-case pre-allocation buffers.

---

### Phase 3 & 4: Targeted Architectural Mutations

#### 1. Enable Prefix / Prompt Caching
Configure the inference engine to cache the KV blocks of the static 8k system prompt grimoire across requests:
```yaml
# Inference Server Config
model: meta-llama/Llama-3-70B-Instruct
enable_prefix_caching: true
block_size: 16
```

#### 2. Enable PagedAttention with FP8 KV-Cache
Convert the KV-cache from FP16 ($16\text{-bit}$) to FP8 ($8\text{-bit}$) with dynamic per-tensor scaling:
```yaml
kv_cache_dtype: fp8
gpu_memory_utilization: 0.90
max_num_seqs: 128
```

#### 3. Bounded Semantic Relaxation Verification (Perplexity Check)
Verify that FP8 KV-cache quantization does not degrade agent reasoning capabilities:
```bash
# Run automated evaluation suite
python run_evals.py --benchmark gsm8k,humaneval --precision fp8_kv
# Result: GSM8K score: 86.4% (FP16 was 86.6%, within acceptable delta eps < 0.3%)
```

---

### Phase 5: Re-measurement & Comparison ($M_{\text{post}}$)
Under the identical 64-user agent workload:

| Metric | Pre-Optimization ($M_{\text{pre}}$) | Post-Optimization ($M_{\text{post}}$) | Delta ($\Delta$) | Status |
| :--- | :--- | :--- | :--- | :--- |
| **TTFT ($P_{99}$)** | $3,450\text{ms}$ | $380\text{ms}$ | **$-89.0\%$** | ✅ Target Met ($< 600\text{ms}$) |
| **ITL ($P_{99}$)** | $48\text{ms/token}$ | $29\text{ms/token}$ | **$-39.5\%$** | ✅ Faster Streaming |
| **Peak GPU VRAM** | $77.8\text{GB}$ | $44.2\text{GB}$ | **$-43.2\%$** | ✅ 35GB Headroom Recovered |
| **Max Concurrency** | $64\text{ streams}$ | $140\text{ streams}$ | **$+118\%$** | ✅ Serving Capacity Doubled |
| **Aggregate Tok/s** | $310\text{ tok/s}$ | $740\text{ tok/s}$ | **$+138\%$** | ✅ Throughput Supercharged |

---

### Phase 6 & 7: Regression Shielding
The agent adds a pre-merge evaluation gate in CI asserting that prompt caching hit rate remains $\ge 90\%$ and FP8 perplexity does not deviate by $> 0.5\%$. The task is certified and signed off.
