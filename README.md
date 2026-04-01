# CLI Best Practices for AI Agents

Most CLIs were built for humans typing at a keyboard. AI agents aren't humans. They can't press arrow keys, answer "are you sure?" prompts, or parse colorful table output. They hallucinate flags that don't exist, retry failed commands in loops, and choke on 55,000-token MCP schemas.

This repo collects what I've learned building two agent-facing CLIs ([craft-cli](https://github.com/nerveband/craft-cli) and [agent-to-bricks](https://github.com/nerveband/agent-to-bricks)), plus patterns from the broader community: articles, benchmarks, open-source tools, and the MCP vs CLI debate that consumed dev Twitter in early 2026.

## What's here

```
principles/          Why these patterns matter
patterns/            How to implement each one
scorecards/          Score your own CLI (0-21 scale)
ecosystem/           The 2026 landscape, repos, articles, benchmarks
```

## The short version

If you're building a CLI that agents will use, these are the things that matter most:

1. **JSON by default.** Not as an option, as the default when piped. Human-readable output is for terminals.
2. **No interactive prompts.** Every input is a flag. Prompts are a fallback, never the primary path.
3. **Structured errors.** `{"error": {"code": "AUTH_FAILED", "message": "...", "hint": "Run: craft config add"}}`, not just "Error: something broke."
4. **`--dry-run` everywhere.** Agents retry. Destructive commands without dry-run are time bombs.
5. **Progressive discovery.** Don't dump your whole schema upfront. `mycli` shows subcommands, `mycli deploy --help` shows flags with examples.
6. **Idempotent operations.** Running the same command twice returns "already done, no-op." Not a duplicate resource.

## The Agent DX CLI Scale

A 7-axis scoring framework (0-21) for evaluating how agent-friendly a CLI is. See [scorecards/](scorecards/) for the full rubric and examples.

| Range | Rating | What it means |
|-------|--------|---------------|
| 0-5 | Human-only | Agents will struggle |
| 6-10 | Agent-tolerant | Works, but wastes tokens and makes avoidable errors |
| 11-15 | Agent-ready | Solid. Structured I/O, validation, some introspection |
| 16-21 | Agent-first | Purpose-built. Full introspection, hardening, safety rails |

## The MCP vs CLI numbers

From [Scalekit's benchmark](https://www.scalekit.com/blog/mcp-vs-cli-use) (75 runs):

- CLI is **10-32x cheaper** per interaction
- MCP had **28% failure rate** (ConnectTimeout); CLI had **0%**
- GitHub's MCP server ships **~55,000 tokens** of tool definitions. `gh` CLI needs roughly zero.
- At 10K agent interactions/day, that's **$500-$2K/month** in token cost difference

The consensus: CLI for the inner loop (speed, tokens, developer control). MCP for the outer loop (multi-tenant auth, governance). Most teams need both.

## Contributing

This is a living document. If you've built an agent-facing CLI and learned something the hard way, PRs are welcome.

## License

MIT
