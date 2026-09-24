# Harness Discovery & Calibration

> **Mandate**: Never assume a fixed test runner or command. Discover the repository's native test harness through manifests and configurations, verify the baseline health before writing new tests, and execute scoped test subsets without running the entire 45-minute suite on every micro-edit.

---

## 1. Automatic Runner Detection

Inspect repository root and workspace manifests in this exact sequence:

| Ecosystem / Files | Detected Harness | Preferred Scoped Execution Command | Full Suite Execution Command |
| :--- | :--- | :--- | :--- |
| **Node / TS** (`package.json`) | Vitest | `npx vitest run path/to/test.ts` | `npm test` or `pnpm test` |
| **Node / TS** (`package.json`) | Jest | `npx jest path/to/test.ts -t "test name"` | `npm test` |
| **Node / TS** (`package.json`) | `node:test` (Native) | `node --test path/to/test.mjs` | `node --test` |
| **Python** (`pyproject.toml`, `setup.cfg`) | Pytest | `pytest path/to/test_file.py -k "test_name" -v` | `pytest -v` |
| **Python** (stdlib only) | `unittest` | `python -m unittest tests.test_file.TestClass.test_method` | `python -m unittest discover tests` |
| **Go** (`go.mod`) | `go test` | `go test -v -run TestName ./pkg/...` | `go test -v ./...` |
| **Rust** (`Cargo.toml`) | `cargo test` | `cargo test test_name -- --nocapture` | `cargo test` |
| **Java / Kotlin** (`pom.xml`) | Maven Surefire | `mvn test -Dtest=TestClassName#testMethodName` | `mvn test` |
| **Java / Kotlin** (`build.gradle`) | Gradle | `./gradlew test --tests "com.pkg.TestClass.testMethod"` | `./gradlew test` |
| **.NET / C#** (`*.csproj`) | dotnet test | `dotnet test --filter "FullyQualifiedName=TestClass.Method"` | `dotnet test` |
| **Ruby** (`Gemfile`) | RSpec / Minitest | `bundle exec rspec spec/path_spec.rb:42` | `bundle exec rspec` |

---

## 2. Pre-Flight Baseline Calibration

Before applying code changes or adding new tests:

1. **Run the existing test suite once**:
   - Verify whether all tests currently pass or if there are pre-existing broken tests.
   - Record any baseline failures. **Never take blame for pre-existing broken tests**, but report them clearly in your pre-flight summary.
2. **Inspect Active Flags & Environment Variables**:
   - Check if the harness requires local `.env.test` files or special flags (e.g. `NODE_ENV=test`, `DATABASE_URL=sqlite::memory:`).
   - Check whether test coverage thresholds are enforced in CI configs (`jest.config.js`, `pyproject.toml`).

---

## 3. Fast Scoped Execution vs Full Suite Run

* **During Active Development**: Run ONLY the specific test file or test case related to the function under edit (e.g. `pytest tests/test_auth.py -k test_token_expiration`). This keeps iteration fast and token usage low.
* **Before Task Completion**: Run the full workspace test suite (or package-scoped test suite in a monorepo) to verify zero regressions across existing features.
