# Scorecard: craft-cli

**Date:** 2026-04-01
**Version:** v1.9.0
**Agent CLI Audit: 45/50 (Agent-first)**
**Agent DX Scale: 14/21 (Agent-ready)**

A Go CLI for [Craft.do](https://craft.do) documents by [Ashraf Ali](https://github.com/nerveband). 30+ commands covering documents, blocks, folders, tasks, collections, whiteboards, comments, uploads, and search.

Repo: [nerveband/craft-cli](https://github.com/nerveband/craft-cli)

## Agent CLI Audit (50-check, [full spec](agent-cli-audit.md))

| Category | Score | Notes |
|----------|-------|-------|
| Discoverability | 7/7 | Progressive help, `craft schema` for JSON introspection, examples everywhere |
| Structured output | 6/6 | JSON default, `--json-errors`, meaningful exit codes, `--quiet` |
| Input flexibility | 5/5 | All flags, `--json`/`--stdin` on blocks+whiteboards, `--api-key` flag |
| Safety rails | 4/6 | Structured JSON dry-run on all 14 mutating cmds, `--yes`. Missing: idempotent create |
| Error handling | 5/5 | Hints with retry guidance on all 8 error codes, stderr/stdout separation |
| Context discipline | 3/5 | `--limit`, `--max-depth`, `--output-only`, `--id-only`. Missing: `--count` |
| Predictability | 4/4 | Consistent resource+verb, same flags everywhere |
| Agent knowledge | 5/5 | AGENTS.md with guardrails+pitfalls, `prompts/`, `docs/llm/`, `craft schema` |
| Resilience | 3/4 | Timeouts, auth degradation, retry guidance. Missing: partial failure reporting |
| Distribution | 3/3 | Single Go binary, goreleaser, `craft upgrade` self-update |
| **Total** | **45/50** | |

## Agent DX Scale (7-axis, [source](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/))

| Axis | Score | Notes |
|------|-------|-------|
| Machine-readable output | 2/3 | JSON default, `--json-errors`. No auto-TTY detection or NDJSON streaming. |
| Raw payload input | 1/3 | `--json`/`--stdin` on blocks+whiteboards. Other mutating cmds use flags only. |
| Schema introspection | 2/3 | `craft schema` with flags, types, safety metadata. Not live API-resolved. |
| Context window discipline | 2/3 | `--limit`, `--max-depth`, `--output-only`, `--id-only`. No streaming pagination. |
| Input hardening | 2/3 | Rejects path traversals, query params, control chars, percent-encoding. |
| Safety rails | 2/3 | Structured JSON dry-run on all mutating cmds. No response sanitization. |
| Agent knowledge | 3/3 | AGENTS.md with guardrails, pitfalls, exit codes. `prompts/` folder. `craft schema`. |
| **Total** | **14/21** | |

## Multi-surface readiness

- [x] MCP mode (project-scoped `.mcp.json`)
- [x] Headless auth (profile-based config, `--api-key`/`--api-url` flags)
- [x] Self-update (`craft upgrade` via GitHub Releases)
- [ ] Agent detection (`AI_AGENT` env var auto-switch)

## Upgrade history

| Version | Audit | DX Scale | Key changes |
|---------|-------|----------|-------------|
| v1.8.0 | 38/50 | 10/21 | Baseline. JSON default, dry-run, LLM docs. |
| v1.9.0 | 45/50 | 14/21 | +schema introspection, +whiteboards, +input hardening, +error hints, +--limit/--max-depth, +structured dry-run |
