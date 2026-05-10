# SDD Search Strategy Design Boundary

This change makes code search configurable for SDD code-reading phases while keeping `grep` as the safe default. RAG/MCP search is an optional acceleration path, not a hard dependency.

## Detection contract

`sdd-init` resolves `search_strategy` in this order:

1. Explicit agent/system config wins.
2. `openspec/config.yaml` wins when present.
3. MCP auto-detection MAY enable `hybrid` only when a tool name or description explicitly indicates semantic, vector, or embedding-based code search.
4. If nothing qualifies, the mode is `grep`.

Generic tools named like `code_search` are not enough by themselves; their description must indicate embedding-backed semantic search to avoid false positives.

## Fallback behavior

| Situation | Behavior |
|-----------|----------|
| No `search_strategy` config | Use `grep` mode. |
| `mode: grep` | Use the existing Grep + targeted Read flow. |
| `mode: hybrid` with valid RAG tool | Query RAG first, then fall back to grep if results are insufficient. |
| RAG tool missing, down, or timing out | Log the issue, use grep for the rest of the phase, and report the risk. |
| Optional reindex fails after `sdd-apply` | Continue; reindex is non-blocking. |

## Why `grep` remains the safe default

`grep` is deterministic, local, and already supported by every SDD phase. Keeping it as the default preserves backwards compatibility and avoids making SDD depend on optional MCP infrastructure.

`hybrid` mode is useful when available because semantic search can reduce exploration work on large codebases, but correctness still comes from the fallback cascade: RAG → grep → targeted read.

## Baseline measurement on this repository

Measured locally on `gentle-ai` using a grep-style scan over tracked UTF-8 files.

| Scenario | Scope | Files scanned | Lines scanned | Matches | Time |
|----------|-------|---------------|---------------|---------|------|
| Find orchestrator forwarding contract (`Search Strategy Forwarding`) | `internal/assets/*/sdd-orchestrator.md` | 11 | 2,944 | 11 | 1.768 ms |
| Find search strategy contract (`search_strategy`) | `internal/assets/skills/**` | 33 | 4,532 | 23 | 3.102 ms |
| Find fallback wording (`fall back to grep`) | `internal/assets/skills/**` | 33 | 4,532 | 1 | 1.850 ms |

These numbers are intentionally a baseline, not a promise of RAG performance. They show the current grep path is cheap for this repository slice, while documenting the exact metrics future `hybrid` runs should compare against: files scanned, lines read, matching confidence, fallback activation, and elapsed time.

## Review checklist

- [ ] `search_strategy` detection is explicit and conservative.
- [ ] Missing or broken RAG never blocks SDD execution.
- [ ] `grep` remains the default when config is absent.
- [ ] Orchestrators forward search config to `sdd-explore`, `sdd-apply`, and `sdd-verify`.
- [ ] Tests cover the contract so future asset edits cannot silently remove it.
