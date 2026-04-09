# SDD Init Details

## Testing Capability Checklist

- Test runner: `package.json` scripts/deps, `pyproject.toml`, `pytest.ini`, `go.mod`, `Cargo.toml`, `Makefile`.
- Test layers: unit runner; integration libraries (`testing-library`, `httpx`, `httptest`, `WebApplicationFactory`); E2E tools (`playwright`, `cypress`, `selenium`, `chromedp`).
- Coverage: `vitest --coverage`, `jest --coverage`, `c8`, `pytest-cov`, `go test -cover`, `coverlet`.
- Quality: linter, type checker, formatter commands.

## Search Strategy Contract

Resolve `search_strategy` using the same first-match-wins shape as Strict TDD:

1. Agent/system marker: explicit `search_strategy` wins.
2. `openspec/config.yaml`: explicit `search_strategy` wins.
3. MCP auto-detection: enable `hybrid` only when a tool name/description explicitly mentions semantic, vector, or embedding-backed code search.
4. Fallback: `grep` is the silent default; do not prompt the user.

Generic names such as `code_search` are not enough unless the description indicates embeddings. Persist the resolved block alongside project context/config:

```yaml
search_strategy:
  mode: grep # or hybrid
  # rag:
  #   mcp_tool: "{semantic_search_tool}"
  #   reindex_tool: "{optional_reindex_tool}"
```

## Skill Registry Scan Rules

- Scan user skills: `~/.claude/skills/`, `~/.config/opencode/skills/`, `~/.gemini/skills/`, `~/.cursor/skills/`, `~/.copilot/skills/`, and the parent directory of this skill file.
- Scan project skills: `{project-root}/.claude/skills/`, `{project-root}/.gemini/skills/`, `{project-root}/.agent/skills/`, and `{project-root}/skills/`.
- Skip `sdd-*`, `_shared`, and `skill-registry`; deduplicate by skill name, preferring project-level skills over user-level skills.
- Read each selected `SKILL.md`; if it exceeds 200 lines, focus on frontmatter plus Critical Patterns / Rules sections.
- Extract `name`, trigger text from `description`, full `SKILL.md` path, and compact rules.
- Generate compact rules as 5-15 actionable lines per skill: constraints, key patterns, breaking changes, and gotchas only. Do not include purpose, motivation, installation steps, full examples, or fluff.
- Scan project convention files: `agents.md`, `AGENTS.md`, project-level `CLAUDE.md`, `.cursorrules`, `GEMINI.md`, and `copilot-instructions.md`.
- For index files such as `AGENTS.md`, extract referenced file paths and include both the index and referenced files in the registry.

## LLM-First Skill Criteria

- Treat skills as runtime instruction contracts, not human documentation.
- Required structure: frontmatter, Activation Contract, Hard Rules, Decision Gates, Execution Steps, Output Contract, References.
- Keep `description` quoted, one physical line, trigger-first, and no longer than 250 characters.
- Target 180-450 body tokens; move examples, schemas, edge cases, and background into local `references/` or `assets/`.
- References must be local files and stable relative to the skill directory when possible.
- Quality gates: hard rules are observable, decision gates cover real forks, output contract states exactly what to return, and references resolve locally.

## Engram Saves

```text
mem_save title/topic_key: sdd-init/{project}
type: architecture
content: detected project context markdown
capture_prompt: false when available

mem_save title/topic_key: sdd/{project}/testing-capabilities
type: config
content: testing capabilities markdown
capture_prompt: false when available

mem_save title/topic_key: skill-registry
type: config
content: registry markdown
capture_prompt: false when available
```

## OpenSpec Skeleton

```text
openspec/
├── config.yaml
├── specs/
└── changes/
    └── archive/
```

`config.yaml` should include concise context, `strict_tdd`, `search_strategy`, testing capabilities, and phase rules for proposal/spec/design/tasks/apply/verify/archive. Keep `context:` under 10 lines.

## Testing Capabilities Format

```markdown
## Testing Capabilities

**Strict TDD Mode**: {enabled/disabled}
**Detected**: {date}

### Test Runner
- Command: `{command}`
- Framework: {name}

### Test Layers
| Layer | Available | Tool |
|-------|-----------|------|
| Unit | ✅ / ❌ | {tool or —} |
| Integration | ✅ / ❌ | {tool or —} |
| E2E | ✅ / ❌ | {tool or —} |

### Coverage
- Available: ✅ / ❌
- Command: `{command or —}`

### Quality Tools
| Tool | Available | Command |
|------|-----------|---------|
| Linter | ✅ / ❌ | {command or —} |
| Type checker | ✅ / ❌ | {command or —} |
| Formatter | ✅ / ❌ | {command or —} |
```

## Output Templates

For each mode, include project, stack, persistence, Strict TDD Mode, Search Strategy, Testing Capabilities table, artifacts created/saved, limitations where relevant, and next steps. Engram mode must mention local/non-shareable limitations; none mode must recommend enabling persistence.
