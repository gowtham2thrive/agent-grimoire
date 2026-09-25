# Technology Radar Template (`TECH_RADAR.md`)

Use this standardized template to document, track, and govern the macro lifecycle of languages, frameworks, libraries, and tools across your project. Store in repository documentation or root as `TECH_RADAR.md`.

---

# Project Technology Radar

> **Governance Policy**: Technologies in this radar are evaluated across 4 lifecycle rings. New features must strictly select components from the **ADOPT** ring. Anything in the **HOLD** ring is frozen against new usage and actively scheduled for retirement.

```
                  ┌────────────────────────────────────────┐
                  │                 HOLD                   │
                  │   ┌────────────────────────────────┐   │
                  │   │             ASSESS             │   │
                  │   │   ┌────────────────────────┐   │   │
                  │   │   │         TRIAL          │   │   │
                  │   │   │   ┌────────────────┐   │   │   │
                  │   │   │   │     ADOPT      │   │   │   │
                  │   │   │   └────────────────┘   │   │   │
                  │   │   └────────────────────────┘   │   │
                  │   └────────────────────────────────┘   │
                  └────────────────────────────────────────┘
```

---

## Ring 1: ADOPT (Default Standards)
*These technologies represent the proven, standard foundation of the repository. Use with zero additional justification required.*

| Category | Technology | Version Baseline | Rationale & Scope |
| :--- | :--- | :--- | :--- |
| **Language** | TypeScript | `^5.4` | Strict type safety, fast build times, universal ecosystem support. |
| **Framework** | Fastify | `^4.26` | High-throughput HTTP routing, schema validation, low overhead. |
| **Testing** | Vitest | `^1.5` | Fast ESM-native execution, compatible with Jest assertions. |
| **Storage** | PostgreSQL | `>= 15.0` | Primary ACID relational store. |

---

## Ring 2: TRIAL (Controlled Production Pilots)
*Vetted candidates running in bounded, low-risk production features to evaluate ergonomics and reliability.*

| Category | Technology | Target Scope | Evaluation Horizon | Sponsoring Team |
| :--- | :--- | :--- | :--- | :--- |
| **Telemetry** | OpenTelemetry SDK | Internal logging microservice | Q3 Evaluation Review | Platform / Ops |
| **Utility** | Zod (v3.23) | API gateway input validation | Q3 Evaluation Review | Backend Core |

---

## Ring 3: ASSESS (Exploratory Sandbox Spikes)
*Under active investigation in prototype branches. Strictly prohibited from production deployments.*

| Category | Technology | Research Objective | Prototype PR / Branch |
| :--- | :--- | :--- | :--- |
| **Language** | Rust (WASM) | High-performance client-side image resizing | `spike/wasm-resizer` |

---

## Ring 4: HOLD (Active Sunsetting & Deprecation)
*Frozen: No new adoption allowed under any circumstances. Existing usages must follow staged deprecation.*

| Technology | Obsolescence Signal | Recommended Successor | Deprecation Milestone | Retirement Ticket |
| :--- | :--- | :--- | :--- | :--- |
| **Moment.js** | Upstream deprecated; excessive bundle weight | `date-fns` / Native `Intl` | Version 3.0 Release | `[DEBT-014]` |
| **Node 16** | End of Life reached upstream | Node 20 LTS | Immediate | `[DEBT-018]` |
| **Request** | Library completely unmaintained | `undici` / native `fetch` | End of Q2 | `[DEBT-022]` |
