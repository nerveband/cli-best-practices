# CLI Best Practices for AI Agents

Most CLIs were built for humans typing at a keyboard. AI agents aren't humans. They can't press arrow keys, answer "are you sure?" prompts, or parse colorful table output. They hallucinate flags that don't exist, retry failed commands in loops, and choke on 55,000-token MCP schemas.

This repo collects what I've learned building 10+ agent-facing CLIs, plus patterns from the broader community: articles, benchmarks, open-source tools, and the MCP vs CLI debate that consumed dev Twitter in early 2026.

By [Ashraf Ali](https://github.com/nerveband).

## What's here

```
principles/          Why these patterns matter
patterns/            How to implement each one
scorecards/          Score your own CLI (0-50 scale)
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

## Scoring frameworks

### Agent CLI Audit (50 checks, original)

A runnable, pass/fail test spec I created for evaluating CLI agent-friendliness. Unlike the Agent DX Scale (which is a subjective 0-3 rubric), this is a concrete checklist where each item can be verified by running a command and checking the output. An agent can score any CLI in under 5 minutes. See [scorecards/agent-cli-audit.md](scorecards/agent-cli-audit.md).

10 categories, 50 checks: discoverability, structured output, input flexibility, safety rails, error handling, context window discipline, predictability, agent knowledge, resilience, and distribution.

### Agent DX CLI Scale (0-21)

Justin Poehnelt's 7-axis scoring framework from ["Rewrite Your CLI for AI Agents"](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/). Good for quick assessment but subjective. See [principles/agent-dx-scale.md](principles/agent-dx-scale.md).

## The MCP vs CLI numbers

From [Scalekit's benchmark](https://www.scalekit.com/blog/mcp-vs-cli-use) (75 runs):

- CLI is **10-32x cheaper** per interaction
- MCP had **28% failure rate** (ConnectTimeout); CLI had **0%**
- GitHub's MCP server ships **~55,000 tokens** of tool definitions. `gh` CLI needs roughly zero.
- At 10K agent interactions/day, that's **$500-$2K/month** in token cost difference

The consensus: CLI for the inner loop (speed, tokens, developer control). MCP for the outer loop (multi-tenant auth, governance). Most teams need both.

## CLIs I've built (and tested these patterns on)

These aren't theoretical recommendations. Every pattern in this repo was tested on real CLIs in production.

| CLI | Language | What it does | Agent DX |
|-----|----------|-------------|----------|
| [craft-cli](https://github.com/nerveband/craft-cli) | Go | Craft.do document management. 30+ commands, whiteboards, schema introspection. | 45/50 audit, 14/21 DX |
| [agent-to-bricks](https://github.com/nerveband/agent-to-bricks) | Go | Bricks Builder bridge for AI agents. 56 commands, contract-first design. | 9/21 DX (targeting 16) |
| [beeper-api-cli](https://github.com/nerveband/beeper-api-cli) | Go | Cross-platform messaging (WhatsApp, Telegram, Signal, etc.) | |
| [yt-api-cli](https://github.com/nerveband/yt-api-cli) | Go | YouTube Data API v3. Manage videos, playlists, uploads. | |
| [mochi-cli](https://github.com/nerveband/mochi-cli) | Go | Mochi.cards flashcard management. Built for LLM automation. | |
| [cloak-agent](https://github.com/nerveband/cloak-agent) | Go + TS | Stealth browser automation for AI agents. | |
| [ai-happy-design](https://github.com/nerveband/ai-happy-design) | Go | Figma CLI for AI agents. | |
| [zpick](https://github.com/nerveband/zpick) | Go | Single-keypress zmosh session launcher. | |
| [drafts-applescript-cli](https://github.com/nerveband/drafts-applescript-cli) | Go | Drafts app CLI (fork). | |
| [image_sense](https://github.com/nerveband/image_sense) | Python | AI image processing with EXIF metadata writing. | |
| [token-vision](https://github.com/nerveband/token-vision) | Python | Offline image token calculator for AI models. | |

## Original work in this repo

- **[Agent CLI Audit](scorecards/agent-cli-audit.md)** (original): 50-check executable test spec. Combines and extends patterns from all sources listed below into a single runnable framework.
- **[craft-cli scorecard](scorecards/craft-cli.md)** (original): Real-world audit result showing the framework applied to a production CLI.
- **Patterns docs** (original synthesis): Each pattern doc combines learnings from my CLIs with community best practices, cited inline.

## Sources and citations

Everything in this repo cites its sources. Here's the full list:

### Articles and posts

| Title | Author | Source |
|-------|--------|--------|
| [Rewrite Your CLI for AI Agents](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/) | Justin Poehnelt | Blog |
| [Building CLIs for agents](https://x.com/ericzakariasson/status/2036762680401223946) | Eric Zakariasson (Cursor) | X/Twitter |
| [Ship Types, Not Docs](https://shiptypes.com/) | Boris Tane (Cloudflare) | Essay |
| [Patterns for AI Agent Driven CLIs](https://www.infoq.com/articles/ai-agent-cli/) | InfoQ | Article |
| [Designing a CLI Skills Protocol](https://cliwatch.com/blog/designing-a-cli-skills-protocol) | CLIWatch | Blog |
| [MCP is dead. Long live the CLI](https://ejholmes.github.io/2026/02/28/mcp-is-dead-long-live-the-cli.html) | Eric Holmes | Blog |
| [MCP vs CLI: Benchmarking Cost and Reliability](https://www.scalekit.com/blog/mcp-vs-cli-use) | Scalekit | Benchmark |
| [Writing CLI Tools Agents Want to Use](https://dev.to/uenyioha/writing-cli-tools-that-ai-agents-actually-want-to-use-39no) | Uche Enyioha | Dev.to |
| [MCP vs CLI for AI-native development](https://circleci.com/blog/mcp-vs-cli/) | CircleCI | Blog |
| [Your MCP Server Is Eating Your Context Window](https://www.apideck.com/blog/mcp-server-eating-context-window-cli-alternative) | Apideck | Blog |
| [10 Design Principles for Delightful CLIs](https://www.atlassian.com/blog/it-teams/10-design-principles-for-delightful-clis) | Atlassian | Blog |
| [CLI Design Guidelines](https://www.thoughtworks.com/insights/blog/engineering-effectiveness/elevate-developer-experiences-cli-design-guidelines) | Thoughtworks | Blog |
| [CLI UX: Progress Display Patterns](https://evilmartians.com/chronicles/cli-ux-best-practices-3-patterns-for-improving-progress-displays) | Evil Martians | Blog |
| [skill.md explained](https://www.gitbook.com/blog/skill-md) | GitBook | Blog |
| [BetterCLI.org](https://bettercli.org/) | BetterCLI | Reference |
| [GitLab CLI agent enhancement](https://gitlab.com/gitlab-org/cli/-/work_items/8177) | GitLab | Issue |

### Repos and tools

| Repo | What |
|------|------|
| [cli-guidelines/cli-guidelines](https://github.com/cli-guidelines/cli-guidelines) | Canonical CLI design guide (clig.dev). 3,573 stars. |
| [HKUDS/CLI-Anything](https://github.com/HKUDS/CLI-Anything) | Auto-generates CLIs from source code. 13K+ stars. |
| [larksuite/cli](https://github.com/larksuite/cli) | 200+ commands, 19 agent skills. Reference implementation. |
| [agentdx/agentdx](https://github.com/agentdx/agentdx) | MCP server linter. 30 rules, 0-100 score. |
| [jaredpalmer/mogcli](https://github.com/jaredpalmer/mogcli) | Agent-friendly M365 CLI. |
| [brwse/earl](https://github.com/brwse/earl) | AI-safe CLI with HCL template security. |
| [@vercel/detect-agent](https://www.npmjs.com/package/@vercel/detect-agent) | AI agent runtime detection. |

## Contributing

This is a living document. If you've built an agent-facing CLI and learned something the hard way, PRs are welcome.

## License

MIT
