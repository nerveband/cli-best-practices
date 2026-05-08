---
name: cli-best-practices-audit
description: Audit a command line tool for AI-agent usability, agent-native workflows, structured output, safety rails, introspection, async recovery, profiles, artifacts, API payload ergonomics, local data layers, proof gates, and skill packaging. Use when asked to review, score, improve, or harden a CLI for coding agents or automation.
---

# CLI Best Practices Audit

Use this skill to evaluate a CLI that an AI agent will call from a shell. The main checker is [scorecards/agent-cli-audit.md](scorecards/agent-cli-audit.md).

## Fast Workflow

1. Identify the binary and build it if needed.
2. Run `$CLI --help`, `$CLI version` or `$CLI --version`, and one safe read command.
3. Run the 85-point audit in `scorecards/agent-cli-audit.md`.
4. Use `--dry-run`, `--local`, test fixtures, or mocked credentials for mutating commands.
5. Record pass/fail for each check with one evidence command or observation.
6. Produce a category table and a prioritized fix list.

## What To Look For First

- JSON on stdout and structured errors on stderr.
- Non-interactive execution: no prompts in non-TTY contexts.
- One canonical vocabulary: `list`, `get`, `create`, `update`, `delete`; one JSON flag; one confirmation/commit convention.
- Three-layer introspection: human help, versioned `agent-context` or schema JSON, and a task-oriented `SKILL.md`.
- Safe retries: idempotent mutations, `--dry-run`, explicit destructive commitment, async `--wait`, and a durable `jobs` ledger.
- Persistent configuration: profiles, documented precedence, redacted config source inspection.
- Artifact routing and feedback: `--deliver stdout|file:<path>|webhook:<url>` where relevant, plus local feedback capture.
- API-native ergonomics: resource-based command mapping, separate data/error formats, transforms, and first-class file arguments with explicit encodings.
- Domain depth: local sync/search for high-gravity resources, compound insight commands, provenance, competitor feature coverage, and proof-of-behavior gates.
- Contract discipline: schema/codegen or CI validation that prevents docs, skills, and command behavior from drifting.

## Reporting Template

```markdown
# Agent CLI Audit: <CLI>

Binary tested: `<path-or-command>`
Version: `<version>`
Date: `<YYYY-MM-DD>`

| Category | Score |
|----------|-------|
| Discoverability | /7 |
| Structured output | /6 |
| Input flexibility | /5 |
| Safety rails | /6 |
| Error handling | /7 |
| Context discipline | /5 |
| Predictability | /7 |
| Agent knowledge | /7 |
| Resilience | /7 |
| Distribution | /5 |
| Three-layer introspection | /5 |
| Persistent identity/config | /5 |
| Two-way I/O/artifacts | /5 |
| Contract/generation discipline | /5 |
| Unix composability/restraint | /5 |
| API-native payload ergonomics | /5 |
| Domain depth/proof gates | /5 |
| Total | /85 |

## Highest-impact fixes

1. <fix> - <why it matters> - <checks gained>
2. <fix> - <why it matters> - <checks gained>
3. <fix> - <why it matters> - <checks gained>
4. <fix> - <why it matters> - <checks gained>
5. <fix> - <why it matters> - <checks gained>

## Evidence notes

- `<command>` -> <observed behavior>
- `<command>` -> <observed behavior>
```

## Source Guidance

Use and cite the repo's source list when recommending non-obvious changes:

- Trevin Chow's "10 Principles for Agent-Native CLIs" for table-stakes vs. compounding checks, three-layer introspection, async ledgers, profiles, delivery, and feedback.
- Cloudflare's "Building a CLI for all of Cloudflare" for schema-layer vocabulary enforcement, consistent `--json`, local/remote signaling, and local explorer APIs.
- `heygen-com/heygen-cli` for JSON-first behavior, request/response schemas, non-interactive auth, async `--wait`, stable exit codes, and bundled `SKILL.md`.
- `openai/openai-cli` for resource-based API command structure, independently configurable success/error formats, JSONL/raw/YAML output modes, GJSON-style transforms, `@file` payload expansion, explicit `@file://` and `@data://` encodings, and debug-log secret warnings.
- `mvanhorn/cli-printing-press` for local SQLite/FTS sync, `--data-source` control, compound insight commands, competitor feature absorption, provenance manifests, dogfood/proof-of-behavior gates, auth doctor, and anti-gaming rules.
- The Hacker News discussion for safety counterpoints: avoid normalizing careless `--force`, consider `--yes` or `--commit`, preserve Unix composability, and prefer a skill-as-manpage over a separate agent-only CLI.

Do not run destructive commands without a dry-run, local fixture, or explicit user approval.
