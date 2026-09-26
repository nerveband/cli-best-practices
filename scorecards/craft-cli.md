# Scorecard: craft-cli

A Go CLI for [Craft.do](https://craft.do) by [Ashraf Ali](https://github.com/nerveband), covering documents, blocks, folders, tasks, collections, whiteboards, comments, uploads, and search over Craft's REST API and MCP server.

Repo: [nerveband/craft-cli](https://github.com/nerveband/craft-cli)

## Status

| | |
|---|---|
| Latest release | v1.12.0 |
| Last full score | v1.9.0 (2026-04-01): 45/50 legacy audit, 14/21 DX scale |
| Next step | Full re-audit of v1.12.0 against audit v3 (85 checks) |

The v1.9.0 scores below are kept as history. Since then craft-cli added `--count`, `jsonl`/`yaml`/`raw` output, `--fields`, `--transform`, `--deliver`, profiles, a local feedback log, MCP escalation, and a built-in `craft audit agent-dx` command. That built-in command checks mostly whether flags exist, so it isn't a substitute for a behavioral audit.

## Open findings for the v3 re-audit

Found in a static review of v1.12.0 source on 2026-09-26, and checked against the live [Craft Connect API docs](https://connect.craft.do/link/HHRuPxZZTJ6/docs/v1). None of these required live API calls. The re-audit should confirm their behavior.

| Audit check | Finding | Evidence |
|---|---|---|
| 14.2 Contract validation | Document search sends `folderIDs` and `documentIDs`; the current API docs specify `folderIds` and `documentIds`. A search scoped to folders or documents may silently lose its filter. | `internal/api/client.go` (`SearchDocumentsAdvanced`) |
| 10.3 Contract mismatch | The pinned REST contract in `docs/contracts/` predates collection views, reminders, and whiteboards in the live API. No scheduled diff exists. | `docs/contracts/craft-rest-space-openapi.json` |
| 4.6 Safety metadata | `craft schema` infers safety from the leaf command name instead of per-command declarations. | `inferSafety` in `cmd/schema.go` |
| 9.2 Partial failure | Several write helpers discard the per-item results the API returns. | `DeleteDocuments`, `DeleteBlocks` in `internal/api/client.go` |
| 9.3 Retry guidance | HTTP 429 becomes a generic "rate limit exceeded. Retry later"; `Retry-After` and the rate-limit headers are dropped. | `internal/api/client.go` error mapping |
| 3.3 Secrets without argv | A global `--api-key` flag accepts the key on argv. Profiles and `--api-key-env` already provide safer paths, so this check likely passes, but the docs should lead with those. | `cmd/root.go` |
| 5.2 and 15.x Non-interactive | With no saved profile, commands offer interactive setup: a banner on stdout and a read from stdin, with no TTY check. `craft setup` itself has no TTY guard either. | `checkFirstRun` and `runSetup` in `cmd/setup.go` |
| 5.x Error handling (MCP) | MCP tool results with `isError: true` aren't treated as failures, so a failed MCP call can exit 0. | `internal/mcp/client.go` checks only JSON-RPC errors |
| 4.2 and 4.4 Dry-run and commitment | `craft mcp call` ignores the global `--dry-run` and doesn't require `--yes` for `craft_write`, while `craft mcp batch` does both. A write through the generic call runs immediately. | `cmd/mcp.go` (only the batch path checks `isDryRun` and `yesFlag`) |
| 8.7 Docs validated | LLM docs still route collection views to MCP, though the current REST API supports them. | `docs/llm/README.md` |
| 6.4 Count | `list --count` reads a `Total` field from the list response; the static audit found the current list schema doesn't document one. Confirm whether the count is correct. | `cmd/list.go` |

Full detail, including the Craft API gap table: `docs/plans/2026-09-26-api-and-best-practices-audit.md` in the craft-cli repo.

## History: v1.9.0 (2026-04-01)

### Agent CLI Audit, legacy 50-check version

| Category | Score | Notes |
|----------|-------|-------|
| Discoverability | 7/7 | Progressive help, `craft schema` for JSON introspection, examples everywhere |
| Structured output | 6/6 | JSON default, `--json-errors`, meaningful exit codes, `--quiet` |
| Input flexibility | 5/5 | All flags, `--json`/`--stdin` on blocks and whiteboards, `--api-key` flag |
| Safety rails | 4/6 | Structured JSON dry-run on all 14 mutating commands, `--yes`. Missing: idempotent create |
| Error handling | 5/5 | Hints with retry guidance on all 8 error codes, stderr/stdout separation |
| Context discipline | 3/5 | `--limit`, `--max-depth`, `--output-only`, `--id-only`. Missing: `--count` |
| Predictability | 4/4 | Consistent resource and verb, same flags everywhere |
| Agent knowledge | 5/5 | AGENTS.md with guardrails and pitfalls, `prompts/`, `docs/llm/`, `craft schema` |
| Resilience | 3/4 | Timeouts, auth degradation, retry guidance. Missing: partial failure reporting |
| Distribution | 3/3 | Single Go binary, goreleaser, `craft upgrade` self-update |
| **Total** | **45/50** | |

### Agent DX Scale ([source](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/))

| Axis | Score | Notes |
|------|-------|-------|
| Machine-readable output | 2/3 | JSON default, `--json-errors`. No auto-TTY detection or NDJSON streaming. |
| Raw payload input | 1/3 | `--json`/`--stdin` on blocks and whiteboards. Other mutating commands use flags only. |
| Schema introspection | 2/3 | `craft schema` with flags, types, safety metadata. Not live API-resolved. |
| Context window discipline | 2/3 | `--limit`, `--max-depth`, `--output-only`, `--id-only`. No streaming pagination. |
| Input hardening | 2/3 | Rejects path traversals, query params, control chars, percent-encoding. |
| Safety rails | 2/3 | Structured JSON dry-run on all mutating commands. (Scored under the old level 3, which asked for response sanitization.) |
| Agent knowledge | 3/3 | AGENTS.md with guardrails, pitfalls, exit codes. `prompts/` folder. `craft schema`. |
| **Total** | **14/21** | |

### Upgrade history

| Version | Audit | DX Scale | Key changes |
|---------|-------|----------|-------------|
| v1.8.0 | 38/50 | 10/21 | Baseline. JSON default, dry-run, LLM docs. |
| v1.9.0 | 45/50 | 14/21 | Schema introspection, whiteboards, input hardening, error hints, `--limit`/`--max-depth`, structured dry-run |
| v1.12.0 | Pending | Pending | `--count`, more output formats, `--fields`, `--transform`, `--deliver`, profiles, feedback log, MCP escalation, batching |
