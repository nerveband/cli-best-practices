# ahd-figma (ai-happy-design) CLI Scorecard

**Date:** 2026-04-03
**Version:** 0.12.0
**Binary:** `ahd-figma` (Go, single Mach-O arm64)
**Repo:** [nerveband/ai-happy-design](https://github.com/nerveband/ai-happy-design)

---

## Agent CLI Audit: 50/50 (Best-in-class)

| Category | Score | Max |
|----------|-------|-----|
| 1. Discoverability | 7 | 7 |
| 2. Structured Output | 6 | 6 |
| 3. Input Flexibility | 5 | 5 |
| 4. Safety Rails | 6 | 6 |
| 5. Error Handling | 5 | 5 |
| 6. Context Window Discipline | 5 | 5 |
| 7. Predictability | 4 | 4 |
| 8. Agent Knowledge | 5 | 5 |
| 9. Resilience | 4 | 4 |
| 10. Distribution | 3 | 3 |
| **Total** | **50** | **50** |

---

## Agent DX CLI Scale: 21/21 (Agent-first, max)

| Axis | Score | Evidence |
|------|-------|----------|
| 1. Machine-Readable Output | 3/3 | All commands output JSON. `schema --json` returns 144 schemas as JSON array. NDJSON streaming on batch (`--ndjson`). Auto-compact JSON in non-TTY. Structured errors on stderr with `code`, `error`, `hint`, `retryable`. |
| 2. Raw Payload Input | 3/3 | 8 input methods (positional JSON, `-p` flag, stdin, file, directory, glob, `-o` inline, `-f` file). Auto-normalization with `--no-fix` opt-out. Zero translation loss. Parameter shorthands inside JSON. |
| 3. Schema Introspection | 3/3 | 144 typed command schemas with params, types, required, enums, patterns, min/max, defaults, aliases. Runtime-resolved metadata: `readOnly`, `destructive`, `idempotent`. Fuzzy suggestions for unknown commands. |
| 4. Context Window Discipline | 3/3 | `--fields`, `--limit`, `--id-only`, `--count` on command. `--compact` on batch. `detail` param (minimal/compact/full) on read commands. `--ndjson` streaming. Skill files with field mask guidance. |
| 5. Input Hardening | 3/3 | Control character rejection. Path traversal detection (literal + percent-encoded). Output path sandboxing (rejects `/etc`, `/usr`, system dirs). Prompt injection sanitization (`SanitizeResponseMap`). Documented security posture. |
| 6. Safety Rails | 3/3 | `--dry-run` on both `command` and `batch`. Dedicated `validate` command. `--strict-quality` quality gate. Response sanitization. `commitUndo()` on all plugin writes. `--if-not-exists` for idempotent creation. |
| 7. Agent Knowledge Packaging | 3/3 | AGENTS.md (300+ lines) with guardrails, error codes, exit codes, security posture. SKILL.md with bootstrap commands. 4 MCP prompts. Reference files (design-patterns, batch-examples, css-to-figma). 3-layer discoverability. |

---

## Multi-Surface Readiness

- [x] **MCP (stdio JSON-RPC)** — `ahd-figma mcp` via mcp-go, with 4 design intelligence prompts
- [x] **Extension/plugin install** — `ahd-figma register` auto-configures Claude Code, Claude Desktop, Cursor, Windsurf, VS Code, Zed
- [x] **Headless auth** — env vars (`AHD_CHANNEL`, `PORT`), TOML config, no browser redirect

---

## Detailed Check Results

### Category 1: Discoverability (7/7)
- 1.1 Root help lists 19 subcommands with descriptions
- 1.2 All subcommands respond to `--help`
- 1.3 `examples` command provides 6 categories of realistic payloads
- 1.4 Examples include real hex colors, canvas sizes, gradients, semantic content
- 1.5 Progressive disclosure: `ahd-figma` -> `schema` -> `schema <cmd> --json`
- 1.6 `schema --json`, `actions --json`, `tools --llm --json` all produce valid JSON
- 1.7 `--version` returns version string

### Category 2: Structured Output (6/6)
- 2.1 All commands output valid JSON
- 2.2 Consistent envelope structure across commands
- 2.3 Compact JSON when piped (TTY auto-detection)
- 2.4 Errors: `{"code":"...","error":"...","hint":"...","retryable":bool}`
- 2.5 Non-zero exit codes on error (differentiated: 1/2/3/4)
- 2.6 `--quiet/-q` suppresses all non-data output

### Category 3: Input Flexibility (5/5)
- 3.1 All inputs via flags/positional args, no interactive prompts
- 3.2 Stdin: `cat ops.json | ahd-figma batch`
- 3.3 Env vars: `AHD_CHANNEL`, `PORT`
- 3.4 Both positional and flag-based input supported
- 3.5 Raw JSON payload as first-class input

### Category 4: Safety Rails (6/6)
- 4.1 `--dry-run` on `command` validates params against schema
- 4.2 `--dry-run` on `batch` validates all ops + design lint
- 4.3 Dry-run shows auto-fixes (e.g., "red" -> "#FF0000"), blocked/fixed counts
- 4.4 No interactive confirmations (by design)
- 4.5 `--if-not-exists` for idempotent creation
- 4.6 `"destructive": true` / `"readOnly": true` / `"idempotent": true` in schema output

### Category 5: Error Handling (5/5)
- 5.1 Errors include `hint` field with fix instructions
- 5.2 Fail-fast on missing args
- 5.3 Distinct error codes: `TIMEOUT`, `CONNECTION_ERROR`, `NODE_NOT_FOUND`, etc.
- 5.4 `retryable` field: true for TIMEOUT/CONNECTION/NO_PLUGIN, false for others
- 5.5 Errors to stderr, data to stdout

### Category 6: Context Window Discipline (5/5)
- 6.1 `--fields id,name,type` filters response to requested fields
- 6.2 `--limit 5` caps array results
- 6.3 `--id-only` returns `[{"id":"42:1"}]`
- 6.4 `--count` returns `{"count": 47}`
- 6.5 `depth` param on `node.get_tree` (1-20) + `compact:true`

### Category 7: Predictability (4/4)
- 7.1 All 144 commands follow `domain.action` naming
- 7.2 Shared flags consistent across subcommands
- 7.3 Deterministic output (same command = same structure)
- 7.4 Exit codes documented in AGENTS.md (0/1/2/3/4)

### Category 8: Agent Knowledge (5/5)
- 8.1 AGENTS.md at project root (300+ lines)
- 8.2 Guardrails: NEVER/MUST/CRITICAL rules
- 8.3 Multi-step workflow examples
- 8.4 Common mistakes documented (gotchas section + catalog)
- 8.5 Skills shipped: `skills/ai-happy-design/` with SKILL.md + references

### Category 9: Resilience (4/4)
- 9.1 `--timeout` flag (default 300s, configurable)
- 9.2 `--fail-fast` + `--retries` + `--retry-delay-ms` on batch
- 9.3 `retryable` field in all error responses
- 9.4 `status` command shows relay/plugin state gracefully

### Category 10: Distribution (3/3)
- 10.1 Single Go binary, no runtime dependencies
- 10.2 `ahd-figma upgrade` self-update from GitHub releases
- 10.3 CLI-plugin version mismatch detection in `status` output

---

## Standout Features

1. **Schema auto-correction in dry-run** — Color "red" auto-fixed to "#FF0000", enum typos fuzzy-matched, numbers clamped to min/max, CSS named colors resolved
2. **3-layer design intelligence** — `catalog_llm.go` (source of truth) -> `describe.go` (MCP tools) -> `SKILL.md` (agent skill file)
3. **Batch step interpolation** — `${{steps.STEP_NAME.result.id}}` references across batch operations
4. **Output path sandboxing** — Export `-o` restricted to CWD/home/temp, system dirs blocked
5. **commitUndo() on all writes** — Every mutation undoable via Cmd+Z in Figma
6. **4 MCP prompts** — Design strategy, batch workflow, common mistakes, token computation
7. **144 typed schemas** — Every command has full JSON schema with safety metadata
