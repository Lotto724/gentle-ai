---
name: sdd-verify
description: "Trigger: SDD verification phase, verify change. Execute tests and prove implementation matches specs, design, and tasks."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: gentleman-programming
  version: "3.0"
---

## Activation Contract

Run when the orchestrator launches verification for an SDD change. You are the quality gate: prove completion with source inspection plus real execution evidence.

## Hard Rules

- Read proposal, spec, design, and tasks before judging implementation.
- Execute relevant tests; static analysis alone is never verification.
- A spec scenario is compliant only when a covering test passed at runtime.
- Compare specs first, design second, task completion third.
- Use shared SDD Section F for code search; if no `search_strategy` is available, default to grep.
- Do not fix issues; report them for the orchestrator/user.
- Persist `verify-report` according to mode: Engram, openspec file, hybrid both, or inline-only for `none`.
- If Strict TDD is active, load `strict-tdd-verify.md` from this skill directory; if inactive, never load it.
- Return the Section D envelope from `../_shared/sdd-phase-common.md`.

## Decision Gates

| Condition | Action |
|---|---|
| Orchestrator says `STRICT TDD MODE IS ACTIVE` | Treat as authoritative. |
| Cached/config `strict_tdd: true` and runner exists | Strict TDD verify; load module. |
| Strict TDD false or no runner | Standard verify; skip TDD checks. |
| `search_strategy.mode: hybrid` and RAG tool available | Use RAG-first cascade from Section F. |
| No search config or RAG unavailable | Use grep path from Section F. |
| Task incomplete | CRITICAL for core task, WARNING for cleanup task. |
| Test command exits non-zero | CRITICAL. |
| Spec scenario has no passing covering test | CRITICAL `UNTESTED` or `FAILING`. |
| Design deviation exists | WARNING unless it breaks a spec. |

## Execution Steps

1. Load relevant skills via shared SDD Section A.
2. Retrieve artifacts via shared Section B for the active persistence mode.
3. Resolve testing/TDD mode from cached capabilities, config, or project files.
4. Resolve `search_strategy` from orchestrator prompt, project context, or config; use shared Section F for source inspection.
5. Count completed and incomplete tasks.
6. Map each spec requirement/scenario to implementation evidence and tests.
7. Check design decisions against changed code.
8. Run test, build/type-check, and coverage commands when available.
9. Build the behavioral compliance matrix from actual test results.
10. Persist and return the verification report.

## Output Contract

Return `## Verification Report` with change, mode, completeness table, build/tests/coverage evidence, spec compliance matrix, correctness table, design coherence table, issues grouped as CRITICAL/WARNING/SUGGESTION, and final verdict `PASS`, `PASS WITH WARNINGS`, or `FAIL`.

## References

- [references/report-format.md](references/report-format.md) — full report template, compliance statuses, and command evidence fields.
- [strict-tdd-verify.md](strict-tdd-verify.md) — load only when Strict TDD is active.
- `../_shared/sdd-phase-common.md` — skill loading, retrieval, persistence, code search, and return envelope.
