# Human-Computer Interaction (HCI) Mathematical Laws & Heuristics

This reference codifies the foundational mathematical laws and empirical heuristics of human-computer interaction. These principles derive from human biomechanics, cognitive psychology, and information processing limits. They apply across every computing medium.

---

## 1 · The Mathematical HCI Laws

```
┌────────────────────────────────────────────────────────────────────────┐
│                        THE 7 PILLARS OF HCI MATH                       │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Fitts's Law            │ Target Acquisition Time = f(Distance, Width)│
│ 2. Hick-Hyman Law         │ Decision Latency = f(log₂ Choices)         │
│ 3. Miller's Law           │ Working Memory Limit = 7 ± 2 Chunks        │
│ 4. Jakob's Law            │ Expectation Transfer from Other Systems    │
│ 5. Tesler's Law           │ Complexity Conservation (System vs. User)  │
│ 6. Doherty Threshold      │ Interaction Flow Boundary ≤ 400ms          │
│ 7. Postel's Law           │ Liberal Ingestion, Conservative Output     │
└────────────────────────────────────────────────────────────────────────┘
```

---

### 1.1 Fitts's Law: Target Acquisition Ergonomics

#### Mathematical Formulation
The time $MT$ (Movement Time) required to rapidly move to a target area is a logarithmic function of the ratio between the distance to the target ($D$) and the width of the target ($W$):

$$MT = a + b \cdot \log_2\left(\frac{2D}{W}\right) = a + b \cdot ID$$

Where:
* $a, b$: Empirically determined constants based on input device (mouse, capacitive touch, stylus, eye-tracker).
* $ID = \log_2\left(\frac{2D}{W}\right)$: **Index of Difficulty** measured in bits.

#### Engineering Directives
1. **Maximize Target Width ($W$) for High-Frequency Actions**: Primary buttons, common menu items, and critical triggers must have generous hit boundaries.
   * *Mobile/Touch*: Minimum physical dimension $\ge 44 \times 44\text{pt}$ ($9\text{mm}$).
   * *Desktop/Web*: Minimum hit boundary $\ge 32 \times 32\text{px}$ with at least $8\text{px}$ gutter.
2. **Minimize Target Distance ($D$)**: Place contextual actions in immediate proximity to the triggering cursor or thumb rest position (e.g. inline action menus, contextual right-click menus, bottom sheets on mobile).
3. **The Infinite Width of Screen Edges ($W \to \infty$)**: On desktop displays, the screen borders, corners, and taskbars possess functionally infinite width because the cursor cannot overshoot the physical display boundary. Pin high-value anchors (close window, start menu, top app bar) to physical screen perimeters.

---

### 1.2 Hick-Hyman Law: Logarithmic Decision Latency

#### Mathematical Formulation
The time $T$ required for a human to make a decision amongst $n$ mutually exclusive, equiprobable alternatives is logarithmic:

$$T = b \cdot \log_2(n + 1)$$

If choices have unequal probabilities $p_i$, the formula expands to information entropy:

$$T = b \cdot \sum_{i=1}^n p_i \log_2\left(\frac{1}{p_i} + 1\right)$$

#### Engineering Directives
1. **Prune Root-Level Choices**: When presenting menus, configuration flags, or navigation items, never present 20 flat unranked choices. Chunk choices into hierarchical categories ($n \le 7$).
2. **Favor Default Pre-Selection**: Increase $p_i$ of the recommended option to $\approx 0.8$. When a single intelligent default satisfies $80\%$ of users, decision latency drops asymptotically to near-zero.
3. **Progressive Disclosure**: Expose only the primary 3–5 decisions upfront; hide advanced or niche parameters behind an intentional secondary trigger (e.g., `"Advanced Settings"`, `--all`, or a collapsible accordion).

---

### 1.3 Miller's Law: Working Memory Limits

#### Cognitive Principle
The span of immediate human working memory is constrained to **$7 \pm 2$ discrete chunks** of information. When an interface forces a user to remember more than 7 items across steps, cognitive overload occurs, leading to error spikes and task abandonment.

#### Engineering Directives
1. **Semantic Chunking**: Group long strings of alphanumeric data into 3–4 character clusters:
   * Credit card: `4111 2222 3333 4444` (4 chunks) vs `4111222233334444` (16 un-chunked items $\to$ immediate memory failure).
   * Phone number: `+1 (555) 019-2834` vs `+15550192834`.
   * CLI flags: Group related subcommands (`git remote add`, `git remote rm`) rather than 50 flat global flags.
2. **Recognition Over Recall**: Never require a user to memorize a value on Screen A to type it into Screen B. Pass data forward automatically or display an active summary sidebar.

---

### 1.4 Jakob's Law: Convention Transfer & Mental Models

#### Cognitive Principle
Users spend the vast majority of their digital lives on systems other than yours. Consequently, their expectations, muscle memory, and navigation reflexes are conditioned by global standards.

#### Engineering Directives
1. **Preserve Standard Metaphors**:
   * Search bars reside at the top center or top right of an application shell.
   * Shopping carts / notifications reside at the top right.
   * Primary navigation resides on the left sidebar (desktop) or bottom tab bar (mobile).
   * In CLI tools, `--help` and `-h` display usage; `--version` and `-v` display version; exit code `0` signals success.
2. **Avoid Eccentric Novelty**: Never replace a recognized pattern (e.g. a magnifying glass for search, a gear for settings) with an esoteric custom icon or gesture without extreme justification.

---

### 1.5 Tesler's Law: Conservation of Complexity

#### Axiom
Every software system contains a fixed, irreducible amount of complexity. The engineer has only one choice: **who absorbs that complexity**—the software or the human user?

```
┌────────────────────────────────────────────────────────┐
│                   TOTAL SYSTEM COMPLEXITY              │
├────────────────────────────┬───────────────────────────┤
│ Absorbed by the Software   │ Offloaded onto the User   │
│ (Parsers, defaults, smart  │ (Manual typing, ambiguous │
│ state machines, caching)   │ configs, error recovery)  │
└────────────────────────────┴───────────────────────────┘
```

#### Engineering Directives
1. **Absorb Complexity in the System**: Build intelligent auto-detection, schema inferencing, sensible defaults, and forgiving input normalization into the code so the user does not have to configure 15 manual parameters.
2. **Avoid Under-Engineering Disguised as "Simplicity"**: Forcing a user to manually format a date string as `YYYY-MM-DDTHH:MM:SSZ` because the backend parser is brittle is not "simple"—it is an aggressive violation of Tesler's Law.

---

### 1.6 The Doherty Threshold: Cognitive Flow & Latency Boundaries

#### Empirical Threshold
When computer and human interact at a pace where neither waits more than **$400\text{ms}$** for a response, the human enters a state of cognitive flow. Total productivity increases exponentially rather than linearly.

```
0ms                 100ms               400ms                    10,000ms
├─────────────────────┼───────────────────┼─────────────────────────┤
  Instantaneous         Micro-Feedback      Noticeable Delay          Attention Lost
  (Keydown, active)     (Token, pulse)      (Skeleton, progress)      (Background task)
```

#### Latency Invariant Table
| Latency Range | Human Perception | Mandatory System Response |
| :--- | :--- | :--- |
| **$0 - 100\text{ms}$** | Instantaneous | Immediate UI state change, depressed button state, character render. |
| **$100 - 400\text{ms}$** | Barely perceptible hesitation | Micro-feedback: subtle spinner, progress ring, or first streaming token. |
| **$400\text{ms} - 2\text{s}$** | Noticeable delay | Skeleton screen, determinate progress bar, contextual status copy (`"Fetching records..."`). |
| **$2\text{s} - 10\text{s}$** | Loss of mental flow | Percentage progress bar, cancel affordance, background polling. |
| **$> 10\text{s}$** | Context switch / Task abandonment | Decouple into asynchronous job with system notification upon completion. |

---

### 1.7 Postel's Law: The Robustness Principle

#### Engineering Axiom
> *"Be liberal in what you accept, and conservative in what you send."*

#### Engineering Directives
1. **Tolerant Input Parsing**: If an input requires a telephone number, accept `(555) 123-4567`, `5551234567`, `555-123-4567`, or `+1 555 123 4567`. Strip non-numeric characters automatically rather than rejecting the form with a validation error.
2. **Deterministic Output Emission**: When generating data, exports, API responses, or CLI tables, always output strict, standardized, deterministic formats.

---

## 2 · Nielsen's 10 Usability Heuristics (Multi-Platform Mapping)

| # | Heuristic | Core Meaning | Web / Desktop Implementation | CLI / TUI Implementation | AI / Agentic Implementation |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **Visibility of System Status** | Always keep users informed about what is going on through prompt, appropriate feedback. | Progress bars, skeleton screens, active tab indicators, saving pills. | Real-time ANSI spinners, stage logs (`[1/3] Downloading...`), carriage return rewrites. | Streaming token output ($\le 200\text{ms}$), active tool call status (`"Searching docs..."`). |
| **2** | **Match Between System & Real World** | Speak the user's language; follow real-world conventions. | Domain terminology (`"Invoice"`, `"Patient"`), natural date formats (`"Yesterday"`). | Familiar commands (`init`, `push`, `status`), natural file paths. | Natural conversational language; zero unformatted JSON or internal model logits. |
| **3** | **User Control & Freedom** | Provide clear "emergency exits" for mistakes; robust undo. | Reversible undo toasts, accessible back buttons, escape key dialog dismiss. | Cancel via `Ctrl+C`, rollback command (`git reset`), session undo (`u`). | Cancel streaming generation button, step rollback affordance, session reset. |
| **4** | **Consistency & Standards** | Users should not wonder whether different words mean the same thing. | Shared design tokens, standard button placement, uniform keyboard shortcuts. | POSIX flag conventions (`-f`, `--force`), standard exit codes (`0` vs `1`). | Consistent persona tone, uniform structured output schemas across turns. |
| **5** | **Error Prevention** | Eliminate error-prone conditions or check for them before committing. | Disabled invalid actions with explanatory tooltips, input masking, typeahead. | Interactive prompt confirmation for destructive flags (`--force`), dry-run mode (`--dry-run`). | Pre-flight tool execution preview for high-blast-radius operations. |
| **6** | **Recognition Rather than Recall** | Minimize memory load by making objects, actions, and options visible. | Search filters, autocomplete, persistent sidebars, recent items history. | Command tab-completion, interactive fuzzy finders (`fzf`), contextual help menus. | Dynamic prompt suggestions, context pills showing active memory/files. |
| **7** | **Flexibility & Efficiency of Use** | Accelerate interactions for experienced users without baffling novices. | Keyboard shortcuts (`Cmd+K`), batch selection, power-user view toggles. | Shell aliases, pipeable raw stdout (`--json`), non-interactive flags (`-y`). | Keyboard shortcuts for accept/reject, direct slash commands (`/plan`, `/test`). |
| **8** | **Aesthetic & Minimalist Design** | Eliminate irrelevant or rarely needed information that competes with vital signal. | High whitespace, visual hierarchy, progressive disclosure of secondary data. | Clean default terminal output; reserve verbose telemetry for `--verbose` / `-v`. | Direct, concise synthesis; avoid redundant preambles and boilerplate essays. |
| **9** | **Help Users Recognize, Diagnose, & Recover from Errors** | Express errors in plain language, precisely indicate the problem, and suggest a solution. | 3-part error banners (Cause + Impact + Recovery Link) directly at the error site. | Error to `stderr` with actionable remediation command (`"Did you mean: git checkout?"`). | Clear diagnostic of tool failure with self-correction proposal. |
| **10** | **Help & Documentation** | Provide easy-to-search documentation focused on the user's immediate task. | Contextual inline help tooltips, searchable knowledge base, empty state guides. | Man pages, detailed `--help` output with concrete usage examples. | Interactive `/help` command, inline capability descriptions for available tools. |
