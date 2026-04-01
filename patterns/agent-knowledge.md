# Agent knowledge packaging

A CLI can be perfectly structured — JSON output, dry-run, the works — and agents will still use it wrong if they don't know what it can do. The packaging matters.

## Progressive disclosure, not schema dumps

The worst thing you can do is load 55,000 tokens of tool definitions into an agent's context window at conversation start. That's what happens with some MCP servers. The agent's working memory fills up with schema it may never use.

Better approach:

1. Agent sees `mycli` and learns top-level subcommands (~10 lines)
2. Agent picks one: `mycli deploy --help` shows flags and examples (~30 lines)
3. Agent uses it

Total context cost: ~40 lines. Compare to the MCP approach where the agent loads the entire API surface upfront.

## AGENTS.md

Ship an `AGENTS.md` in your repo root. Keep it short — under 100 lines. Include:

- What the CLI does (one paragraph)
- Auth setup (env vars, config commands)
- The most common workflows (3-5 examples)
- Explicit guardrails: "always use --dry-run before delete," "use --fields to limit output size"

The CLIWatch team found that a one-liner in AGENTS.md pointing to a skills subcommand (~40 tokens) eliminated blind guessing and improved GPT-5.2's pass rate from 33% to 50%.

## The `skills` subcommand

An emerging pattern. The CLI exposes a `skills` or `schema` command that returns structured JSON describing what it can do:

```bash
$ mycli skills
{
  "commands": [
    {
      "name": "deploy",
      "description": "Deploy to an environment",
      "flags": ["--env", "--tag", "--dry-run", "--yes"],
      "examples": ["mycli deploy --env staging --tag v1.2.3"],
      "safety": {"destructive": false, "idempotent": true}
    }
  ]
}
```

This is the machine-readable equivalent of `--help`. Agents can parse it programmatically.

## SKILL.md files

A format gaining traction across coding agents (Claude Code, Cursor, Gemini CLI, and others). YAML frontmatter with name, description, and trigger conditions, followed by markdown content:

```yaml
---
name: deploy-workflow
description: Deploy to staging and production
---

1. Run `mycli deploy --env staging --tag TAG --dry-run` to preview
2. If dry-run looks right, run without --dry-run
3. Verify with `mycli status --env staging`
4. When ready for prod, run `mycli deploy --env production --tag TAG --dry-run`
```

Teams that ship SKILL.md files alongside their CLIs report much higher agent success rates than those relying on docs alone. The key is that skills describe workflows, not just individual commands.

## Detect when an agent is calling

The `AI_AGENT` environment variable is emerging as a standard (via Vercel's `@vercel/detect-agent`). When set, your CLI can automatically:

- Switch to JSON output
- Suppress progress bars and spinners
- Use structured error format
- Skip interactive prompts entirely
