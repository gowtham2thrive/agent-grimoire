# Example: Matrix Cross-Platform SDK Pipeline

> **Context**: An open-source systems library or multi-language client SDK distributed across Linux, macOS, and Windows. Requires verification against multiple runtime versions without blowing up CI quotas.

---

## 1 · Tiered Matrix Strategy

To prevent running 30 concurrent jobs on every typo PR:
1. **Pull Requests (Canary Tier)**: Run only Ubuntu LTS + Current Stable Runtime.
2. **Mainline & Releases (Comprehensive Tier)**: Run full matrix across 3 OSes $\times$ 3 Runtime versions = 9 jobs.

```mermaid
flowchart TD
    Event["Pipeline Trigger"] --> BranchCheck{"Is Pull Request?"}
    BranchCheck -->|Yes| Canary["Canary Matrix (1 Job)<br/>• OS: Ubuntu-Latest<br/>• Runtime: Node 20 LTS"]
    BranchCheck -->|No (Main/Tag)| FullMatrix["Full Matrix (9 Jobs)<br/>• OS: [Ubuntu, macOS, Windows]<br/>• Runtime: [Node 18, 20, 22]"]
    
    Canary --> PRPass["Fast Status Check (~2m)"]
    FullMatrix --> Artifacts["Build Cross-Platform Binaries<br/>Generate Checksum Manifest"]
```

---

## 2 · Declarative Matrix Configuration

```yaml
name: cross-platform-sdk-matrix

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

permissions: read-all

jobs:
  test-matrix:
    name: Test (${{ matrix.os }}, Runtime ${{ matrix.runtime }})
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        runtime: ['18', '20', '22']
        # Prune redundant combinations on PRs
        exclude:
          - os: ${{ github.event_name == 'pull_request' && 'macos-latest' || '' }}
          - os: ${{ github.event_name == 'pull_request' && 'windows-latest' || '' }}
    steps:
      - uses: actions/checkout@v4
      - name: Setup Runtime
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.runtime }}
          cache: 'npm'
      - run: npm ci
      - run: npm test

  publish-sealed-package:
    name: Seal Artifacts & Publish (Phase 5)
    if: startsWith(github.ref, 'refs/tags/v')
    needs: [test-matrix]
    runs-on: ubuntu-latest
    permissions:
      contents: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - run: npm run build:all
      - name: Generate Checksums
        run: sha256sum dist/* > SHA256SUMS
      - name: Attest Provenance & Publish
        run: |
          echo "Publishing sealed packages to registry with signed provenance."
```
