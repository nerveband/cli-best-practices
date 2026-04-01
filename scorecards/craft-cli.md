# Scorecard: craft-cli

**Date:** 2026-04-01
**Version:** v1.8.0
**Total: 8/21 (Agent-tolerant)**

A Go CLI for Craft.do documents. 30+ commands covering documents, blocks, folders, tasks, collections, comments, uploads, and search.

## Scores

| Axis | Score | Notes |
|------|-------|-------|
| Machine-readable output | 2/3 | JSON default, `--json-errors`, multiple formats. No auto-detect for non-TTY, no NDJSON. |
| Raw payload input | 1/3 | `--json` and `--stdin` on blocks commands. Most mutating commands use individual flags only. |
| Schema introspection | 0/3 | `--help` text only. `craft llm` gives human-readable reference, not machine-readable schema. |
| Context window discipline | 1/3 | `--output-only`, `--id-only`, `--quiet`, `--raw`. No `--fields` for arbitrary selection, no pagination. |
| Input hardening | 0/3 | Basic type checks. Structured exit codes (0-3). No hallucination-pattern validation. |
| Safety rails | 2/3 | Global `--dry-run` on all mutating commands. Output is human prose, not structured JSON. No response sanitization. |
| Agent knowledge packaging | 2/3 | AGENTS.md, `docs/llm/` folder, `prompts/` folder with session prompts. No per-command skill files, no versioned skill library. |
| **Total** | **8/21** | |

## Multi-surface readiness

- [x] MCP mode (project-scoped `.mcp.json`)
- [x] Headless auth (profile-based config, env var support)
- [ ] Agent detection (`AI_AGENT` env var)

## Strengths

- JSON is the default output format, not an afterthought
- Dry-run coverage is comprehensive (move, blocks, tasks, folders, collections, comments, delete, clear)
- Already has LLM-specific documentation (`docs/llm/`, `craft llm styles`)
- Go binary, single file, no runtime dependencies, cross-platform via goreleaser
- Self-update with background version check notifications

## Gaps

- No `craft schema` command for agents to discover capabilities
- Raw JSON input only on blocks commands, not create/update/tasks/folders
- No `--fields` flag for arbitrary field selection
- No input hardening for hallucinated IDs
- Dry-run output is human prose, not structured JSON
- Whiteboards API surface not implemented (5 endpoints)

## Roadmap to 16/21

| Phase | Change | Point gain |
|-------|--------|------------|
| 1 | `craft schema` command + auto-detect non-TTY | +3 (introspection 0->2, output 2->3) |
| 2 | Raw JSON on all mutating commands + `--fields` | +3 (input 1->3, context 1->2) |
| 3 | Input hardening for hallucination patterns | +2 (hardening 0->2) |
| 4 | Structured dry-run output + skill files | +2 (safety 2->3, knowledge 2->3) |
