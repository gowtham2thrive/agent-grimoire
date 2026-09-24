# Diff Hygiene & Secret Guardrails

> **Mandate**: A pull request or commit diff is a permanent historical artifact. Prevent debugging debris, accidental file mutations, formatting noise, and secret leaks from entering source control.

---

## 1. Forbidden Diff Patterns

Before committing or submitting code, scan the diff for these forbidden patterns:

```text
┌─────────────────────────────────┬─────────────────────────────────┐
│       FORBIDDEN PATTERN         │       DETECTION & REMEDIATION   │
├─────────────────────────────────┼─────────────────────────────────┤
│ 1. Temporary Debug Prints       │ `console.log`, `print()`, `p()` │
│                                 │ REMEDIATE: Remove before commit.│
├─────────────────────────────────┼─────────────────────────────────┤
│ 2. Unintended Line Ending Churn │ Entire file shows as modified   │
│                                 │ due to CRLF vs LF differences.   │
│                                 │ REMEDIATE: Normalize git config.│
├─────────────────────────────────┼─────────────────────────────────┤
│ 3. Accidental File Deletions    │ Deleted configuration files or  │
│                                 │ test assets during refactoring. │
│                                 │ REMEDIATE: Revert via git checkout.
├─────────────────────────────────┼─────────────────────────────────┤
│ 4. Scratch & Build Artifacts    │ `.DS_Store`, `npm-debug.log`,   │
│                                 │ `scratch.py`, compiled binaries.│
│                                 │ REMEDIATE: Add to .gitignore.   │
├─────────────────────────────────┼─────────────────────────────────┤
│ 5. Hardcoded Credentials        │ API keys, AWS secrets, tokens.  │
│                                 │ REMEDIATE: Revoke immediately.  │
└─────────────────────────────────┴─────────────────────────────────┘
```

---

## 2. Contextual Secret Hygiene

Never assume an environment variable or token is safe to write to disk. Follow these three rules:

1. **Test Fixtures**: Never use real API tokens in test files. Use clearly fake dummy strings (e.g. `mock_sample_token_for_testing`).
2. **Local Configuration**: Store local environment settings in `.env.local` or `.env` and verify that these files are listed in `.gitignore`.
3. **Commit Inspection**: Run `git diff --staged` before finalizing any commit to visually inspect all additions for credentials.
