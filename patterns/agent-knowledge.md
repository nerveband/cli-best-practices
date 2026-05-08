# Agent knowledge packaging

A CLI can be perfectly structured (JSON output, dry-run, the works) and agents will still use it wrong if they don't know what it can do. The packaging matters.

## Progressive disclosure, not schema dumps

The worst thing you can do is load 55,000 tokens of tool definitions into an agent's context window at conversation start. That's what happens with some MCP servers. The agent's working memory fills up with schema it may never use.

Better approach:

1. Agent sees `mycli` and learns top-level subcommands (~10 lines)
2. Agent picks one: `mycli deploy --help` shows flags and examples (~30 lines)
3. Agent uses it

Total context cost: ~40 lines. Compare to the MCP approach where the agent loads the entire API surface upfront.

## AGENTS.md

Ship an `AGENTS.md` in your repo root. Keep it short, under 100 lines. Include:

- What the CLI does (one paragraph)
- Auth setup (env vars, config commands)
- The most common workflows (3-5 examples)
- Explicit guardrails: "always use --dry-run before delete," "use --fields to limit output size"

The [CLIWatch team](https://cliwatch.com/blog/designing-a-cli-skills-protocol) found that a one-liner in AGENTS.md pointing to a skills subcommand (~40 tokens) eliminated blind guessing and improved GPT-5.2's pass rate from 33% to 50%.

## The `skills` subcommand

An emerging pattern. The CLI exposes a `skills`, `skill-path`, `agent-context`, or `schema` command that returns structured JSON describing what it can do:

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

Trevin Chow's [10 Principles for Agent-Native CLIs](https://trevinsays.com/p/10-principles-for-agent-native-clis) sharpens this into three layers:

1. Human-shaped `--help` for quick terminal discovery.
2. Versioned, machine-readable `agent-context` or schema output for commands, flags, input types, examples, exit codes, safety metadata, profiles, delivery sinks, and feedback support.
3. Long-form `SKILL.md` files for task workflows, not just command reference.

Cloudflare's [CLI rebuild](https://blog.cloudflare.com/cf-cli-local-explorer/) is the large-scale version: one TypeScript schema generates CLI commands, SDKs, MCP, docs, and skills so these layers don't drift.

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

Teams that ship SKILL.md files alongside their CLIs report much higher agent success rates than those relying on docs alone ([GitBook's explainer](https://www.gitbook.com/blog/skill-md) is a good starting point). The key is that skills describe workflows, not just individual commands.

[heygen-com/heygen-cli](https://github.com/heygen-com/heygen-cli) is a concrete reference. Its root `SKILL.md` gives agents install/auth guidance, key workflows, async `--wait` guidance, request/response schema discovery, and the output contract in one short file.

The [HN discussion of agent-native CLIs](https://news.ycombinator.com/item?id=48052333) is a useful corrective: a skill should behave like a concise manpage for agents. It should teach agents to compose the normal CLI surface, not create a separate agent-only behavior layer that humans and scripts cannot use.

## Detect when an agent is calling

The `AI_AGENT` environment variable is emerging as a standard (via Vercel's [`@vercel/detect-agent`](https://www.npmjs.com/package/@vercel/detect-agent)). When set, your CLI can automatically:

- Switch to JSON output
- Suppress progress bars and spinners
- Use structured error format
- Skip interactive prompts entirely

## Feedback loops

Agents hit CLI friction that maintainers rarely see: a missing enum value, a misleading error, an async timeout that should have resumed, or a flag name the model reasonably guessed. Add a local feedback command:

```bash
$ mycli feedback "the --tier flag rejects enterprise but docs list it as valid"
```

Record JSONL locally by default. If a user configures an upstream endpoint, POST the feedback and surface the endpoint status in structured output. Keep this opt-in and discoverable through `agent-context`.

## Real-world: craft-cli and agent-to-bricks

[craft-cli](https://github.com/nerveband/craft-cli) ships an AGENTS.md with guardrails ("always use --dry-run before delete"), exit code table, and a pitfalls section. It also has `craft schema` for machine-readable command discovery with safety metadata, plus a `prompts/` folder with implement/check/release session prompts for AI coding agents.

[agent-to-bricks](https://github.com/nerveband/agent-to-bricks) takes this further with `bricks schema --validate` (CI-enforced contract validation), `bricks discover` (machine-readable site discovery), and `bricks init` (bootstraps CLAUDE.md and skill files for a new project). The repo ships three session prompts: `prompts/implement.md`, `prompts/check.md`, and `prompts/release.md`.
