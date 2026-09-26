---
name: cli-best-practices
description: Audit a command line tool for AI-agent usability and produce an evidence-backed score with prioritized fixes. Covers structured output, error kinds and exit codes, non-interactive safety, declared command effects and retry safety, bounded output, schema introspection, upstream API contract drift, remote-data trust boundaries, profiles, async jobs, and skill packaging. Use when asked to review, score, improve, or harden a CLI for coding agents, automation, or MCP parity.
license: MIT
---

# CLI Best Practices Audit

Use this skill to evaluate a CLI that an AI agent will call from a shell. The checklist is [scorecards/agent-cli-audit.md](scorecards/agent-cli-audit.md). The external standard it aligns with is [the CLI Spec](https://clispec.dev/).

## Workflow

1. Identify the binary and build it if needed. Record the exact version.
2. Run `$CLI --help`, `$CLI --version` (or `$CLI version`), and `$CLI schema` if it exists. Then run one safe read command.
3. If the CLI publishes a CLI Spec schema, validate it (`clispec score $CLI`, or `make check` in the clispec repo) and record the result.
4. Work through all 85 checks. Record pass, fail, or declared exemption for each, with one evidence command or observation.
5. For anything that writes, use `--dry-run`, local fixtures, a sandbox account, or mocked credentials. Never touch real data without explicit user approval.
6. Produce the report below: category table, top five fixes, evidence notes.

## Scoring rules

- **Behavior beats presence.** A flag that exists but does nothing, or a self-audit command that only checks flag names, does not pass. Run the command and observe the result.
- **Declared exemptions pass; silent gaps fail.** A check that cannot apply (pagination on a single-record command, `--wait` with no async jobs) passes only when the CLI's schema or docs declare why, for example `"cardinality": "single"`. List every exemption in the report.
- **Wrappers answer to their upstream.** For API wrappers, spot-check at least one request parameter name and one response shape against the provider's current docs.

## What to look for first

- Data on stdout, diagnostics on stderr, no ANSI when piped, and an explicit format flag that always wins.
- A fixed set of error kinds with declared exit codes, and a JSON error envelope when JSON is selected.
- No hangs without a TTY: prompts become a refusal naming the bypass flag.
- Every command declares its effects (`read_only`, `idempotent`, `non_idempotent`) explicitly, not inferred from its name.
- Unbounded lists paginate server-side and say when output is partial.
- A `schema` command that works with no auth, config, or network.
- Secrets never required on argv.
- Remote content preserved and labeled, never executed or followed.

## Report template

```markdown
# Agent CLI Audit: <CLI>

Binary tested: `<path-or-command>`
Version: `<version>`
Date: `<YYYY-MM-DD>`
CLI Spec: `<conformant to 0.2 | 0.3 candidate | not published | failed: reason>`

| Category | Score | Exemptions |
|----------|-------|------------|
| Discoverability | /7 | |
| Structured output | /6 | |
| Input flexibility | /5 | |
| Safety rails | /6 | |
| Error handling | /7 | |
| Context discipline | /5 | |
| Predictability | /7 | |
| Agent knowledge | /7 | |
| Resilience | /7 | |
| Distribution | /5 | |
| Three-layer introspection | /5 | |
| Persistent identity/config | /5 | |
| Two-way I/O/artifacts | /5 | |
| Contract/generation discipline | /5 | |
| Unix composability/restraint | /5 | |
| API-native payload ergonomics | /5 | |
| Domain depth/proof gates | /5 | |
| Total | /85 | |

## Highest-impact fixes

1. <fix>: <why it matters> (<checks gained>)
2. <fix>: <why it matters> (<checks gained>)
3. <fix>: <why it matters> (<checks gained>)
4. <fix>: <why it matters> (<checks gained>)
5. <fix>: <why it matters> (<checks gained>)

## Evidence notes

- `<command>`: <observed behavior>
- `<command>`: <observed behavior>
```

## Sources to cite

Cite these when a recommendation is not obvious:

- [The CLI Spec](https://clispec.dev/) for output kinds, declared effects, cardinality, error kinds with exit codes, non-TTY refusal, idempotency keys, and bounded output. v0.2 is frozen; v0.3 is a candidate.
- [clig.dev](https://clig.dev/) for general CLI conventions and human defaults.
- [Agent Skills specification](https://agentskills.io/specification) for `SKILL.md` structure.
- [MCP tools specification](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) for `outputSchema`, `structuredContent`, `isError`, and tool annotations when the CLI also ships an MCP surface.
- The [landscape](ecosystem/landscape-2026.md) for async jobs, profiles, delivery, payload ergonomics, local data layers, and the safety counterpoints from the Hacker News discussion.
