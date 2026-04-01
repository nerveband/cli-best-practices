# Agent DX CLI Scale

A scoring framework for evaluating how well a CLI works with AI agents. Seven axes, each scored 0-3, for a total between 0 and 21.

The framework comes from Justin Poehnelt's ["Rewrite Your CLI for AI Agents"](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/) and has been refined through real-world use on multiple CLI projects.

> Human DX optimizes for discoverability and forgiveness.
> Agent DX optimizes for predictability and defense-in-depth.

## The seven axes

### 1. Machine-readable output

Can an agent parse the output without guessing?

| 0 | Human-only: tables, ANSI colors, prose |
|---|----------------------------------------|
| 1 | `--output json` exists but inconsistent across commands |
| 2 | Consistent JSON everywhere, including error responses |
| 3 | NDJSON streaming for paginated results; JSON is the default in piped contexts |

### 2. Raw payload input

Can an agent send the full API payload directly?

| 0 | Only bespoke flags, no structured input |
|---|----------------------------------------|
| 1 | `--json` or stdin for some commands |
| 2 | All mutating commands accept a JSON payload matching the API schema |
| 3 | Raw payload is first-class. The API schema is the documentation, zero translation needed |

### 3. Schema introspection

Can an agent discover what the CLI accepts at runtime?

| 0 | Only `--help` text |
|---|---------------------|
| 1 | `--help --json` or a `describe` command for some surfaces |
| 2 | Full schema for all commands (params, types, required fields) as JSON |
| 3 | Live schemas reflecting the current API version, including enums and nested types |

### 4. Context window discipline

Does the CLI help agents control response size?

| 0 | Full API responses, no way to limit |
|---|--------------------------------------|
| 1 | `--fields` on some commands |
| 2 | Field masks on all reads, pagination controls |
| 3 | Streaming pagination (NDJSON per page), explicit guidance on field selection |

### 5. Input hardening

Does the CLI defend against agent hallucinations (not typos)?

| 0 | Basic type checks only |
|---|------------------------|
| 1 | Some validation, but no hallucination-specific checks |
| 2 | Rejects path traversals, encoded segments, embedded query params in IDs |
| 3 | All of the above plus output sandboxing and an explicit "agent is untrusted" security posture |

### 6. Safety rails

Can agents preview before acting?

| 0 | No dry-run, no sanitization |
|---|------------------------------|
| 1 | `--dry-run` on some commands |
| 2 | `--dry-run` on all mutating commands |
| 3 | Dry-run plus response sanitization against prompt injection in API data |

### 7. Agent knowledge packaging

Does the CLI ship knowledge in formats agents can consume?

| 0 | Only `--help` and a docs site |
|---|--------------------------------|
| 1 | An AGENTS.md or CONTEXT.md with basic guidance |
| 2 | Structured skill files covering per-command workflows |
| 3 | Versioned skill library with explicit guardrails ("always use --dry-run", "always use --fields") |

## Reading the total

| Range | Rating |
|-------|--------|
| 0-5 | Human-only |
| 6-10 | Agent-tolerant |
| 11-15 | Agent-ready |
| 16-21 | Agent-first |

## Bonus: multi-surface readiness

Not scored, but worth noting:

- **MCP mode** (stdio JSON-RPC): typed invocation, no shell escaping
- **Headless auth**: env vars for credentials, no browser redirect
- **Agent detection**: check for `AI_AGENT` env var (via [`@vercel/detect-agent`](https://www.npmjs.com/package/@vercel/detect-agent)) and auto-switch output format
