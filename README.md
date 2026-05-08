# CLI Best Practices for AI Agents

Most CLIs were built for humans typing at a keyboard. AI agents aren't humans. They can't press arrow keys, answer "are you sure?" prompts, or parse colorful table output. They hallucinate flags that don't exist, retry failed commands in loops, and choke on 55,000-token MCP schemas.

This repo collects what I've learned building 10+ agent-facing CLIs, plus patterns from the broader community: articles, benchmarks, open-source tools, and the MCP vs CLI debate that consumed dev Twitter in early 2026.

By [Ashraf Ali](https://github.com/nerveband).

## What's here

```
principles/          Why these patterns matter
patterns/            How to implement each one
scorecards/          Score your own CLI (0-85 scale, with legacy 0-50 baseline)
ecosystem/           The 2026 landscape, repos, articles, benchmarks
```

## The short version

If you're building a CLI that agents will use, these are the things that matter most:

1. **JSON by default.** Not as an afterthought. Keep data on stdout, diagnostics on stderr, and use one canonical convention such as `--json` or global `--format json`.
2. **No interactive prompts.** Every input is a flag, env var, file, or stdin. Prompts are a human fallback, never the primary path.
3. **Structured errors that teach.** Include `code`, `message`, `hint`, and valid enum values when validation fails.
4. **Explicit mutation boundaries.** Agents retry. Use `--dry-run`, idempotency, and a clear destructive commitment convention such as `--yes`, `--commit`, or ecosystem-standard `--force`.
5. **Bounded output.** Paginate, filter, summarize, and tell the agent how to narrow the next call.
6. **Predictable vocabulary.** Use common verbs and flags consistently, and enforce banned aliases mechanically.
7. **Three-layer introspection.** Provide human help, versioned machine-readable `agent-context`/schema, and task-oriented `SKILL.md`.
8. **Async-aware execution.** `--wait`, `--timeout`, resume commands, and a durable jobs ledger beat making the agent write poll loops.
9. **Persistent profiles.** Let agents reuse named identities/configurations across sessions with documented precedence.
10. **Two-way I/O.** Route artifacts to stdout/files/webhooks and give agents a local feedback channel for friction.
11. **API-native payload ergonomics.** Mirror the API's resource model, support nested payloads, first-class files, explicit encodings, transforms, and separate formatting for data vs. errors.
12. **Domain depth.** For high-gravity APIs, add local sync/search, compound insight commands, provenance, and proof-of-behavior checks. Thin endpoint wrappers are the floor, not the ceiling.

## Scoring frameworks

### Agent CLI Audit (85 checks, original)

A runnable, pass/fail test spec I created for evaluating CLI agent-friendliness. Unlike the Agent DX Scale (which is a subjective 0-3 rubric), this is a concrete checklist where each item can be verified by running a command and checking the output. See [scorecards/agent-cli-audit.md](scorecards/agent-cli-audit.md).

17 categories, 85 checks: discoverability, structured output, input flexibility, safety rails, error handling, context window discipline, predictability, agent knowledge, resilience, distribution, three-layer introspection, persistent identity/config, two-way I/O, contract/generation discipline, Unix composability/restraint, API-native payload ergonomics, and domain depth/proof gates.

This repo also ships [SKILL.md](SKILL.md), a compact entrypoint that agents can install or read before running the audit.

## Use this as an agent skill

Point your coding agent at [SKILL.md](SKILL.md) before auditing a CLI, or install it into the agent's skill system if your harness supports local `SKILL.md` files. The skill tells the agent to:

1. Build or locate the CLI binary.
2. Run root help, version, and one safe read command.
3. Score the CLI with the 85-point audit.
4. Use dry-run, local fixtures, or mocked credentials for mutating checks.
5. Return a category table, evidence notes, and the top five highest-impact fixes.

The skill is intentionally short. It should behave like a manpage for agents: enough workflow guidance to prevent blind guessing, without loading the entire repo into context.

## 2026 agent-native upgrade

The original audit focused on defensive readiness: structured output, non-interactive input, dry-run, bounded responses, and useful errors. This upgrade adds the compounding layer that emerged from newer agent-native CLIs and discussions:

- **Schema-enforced consistency:** vocabulary and flags should be checked mechanically, not left to review.
- **Three-layer introspection:** human help, versioned machine-readable schema/agent-context, and workflow-oriented skills.
- **Async recovery:** `--wait`, timeouts, resume commands, and job ledgers.
- **Persistent identity:** profiles and inspectable config precedence.
- **Two-way I/O:** artifact delivery sinks plus local feedback capture.
- **API-native ergonomics:** file arguments, explicit encodings, transforms, separate data/error formatting.
- **Domain depth:** local sync/search, compound insight commands, provenance, competitor coverage, and proof-of-behavior gates.

Thanks to the projects and discussions that sharpened this version: Trevin Chow's agent-native CLI principles, Cloudflare's CLI/local explorer work, HeyGen CLI, OpenAI CLI, CLI Printing Press, and the Hacker News thread that pushed on safety and Unix composability.

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
| [ai-happy-design](https://github.com/nerveband/ai-happy-design) | Go | Figma CLI for AI agents. 144 commands, schema validation, design intelligence. | **50/50 audit, 21/21 DX** |
| [zpick](https://github.com/nerveband/zpick) | Go | Single-keypress zmosh session launcher. | |
| [drafts-applescript-cli](https://github.com/nerveband/drafts-applescript-cli) | Go | Drafts app CLI (fork). | |
| [image_sense](https://github.com/nerveband/image_sense) | Python | AI image processing with EXIF metadata writing. | |
| [token-vision](https://github.com/nerveband/token-vision) | Python | Offline image token calculator for AI models. | |

## Original work in this repo

- **[Agent CLI Audit](scorecards/agent-cli-audit.md)** (original): 85-check executable test spec with a legacy 50-check baseline. Combines and extends patterns from all sources listed below into a single runnable framework.
- **[craft-cli scorecard](scorecards/craft-cli.md)** (original): Real-world audit result showing the framework applied to a production CLI.
- **[ahd-figma scorecard](scorecards/ahd-figma.md)** (original): First CLI to achieve a perfect 50/50 audit score and 21/21 DX scale.
- **Patterns docs** (original synthesis): Each pattern doc combines learnings from my CLIs with community best practices, cited inline.

## Sources and citations

Everything in this repo cites its sources. Here's the full list:

### Articles and posts

| Title | Author | Source |
|-------|--------|--------|
| [Rewrite Your CLI for AI Agents](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/) | Justin Poehnelt | Blog |
| [10 Principles for Agent-Native CLIs](https://trevinsays.com/p/10-principles-for-agent-native-clis) | Trevin Chow | Blog |
| [Building a CLI for all of Cloudflare](https://blog.cloudflare.com/cf-cli-local-explorer/) | Matt Taylor, Dimitri Mitropoulos, Dan Carter | Blog |
| [OpenAI CLI](https://github.com/openai/openai-cli) | OpenAI | Repo |
| [CLI Printing Press](https://github.com/mvanhorn/cli-printing-press) | Michael VanHorn | Repo |
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
| [Principles for agent-native CLIs](https://news.ycombinator.com/item?id=48052333) | Hacker News | Discussion |

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
| [heygen-com/heygen-cli](https://github.com/heygen-com/heygen-cli) | Agent-first video API CLI with JSON stdout, schemas, async `--wait`, and bundled skill guidance. |
| [openai/openai-cli](https://github.com/openai/openai-cli) | Official OpenAI API CLI with resource commands, data/error format controls, transforms, and first-class file argument expansion. |
| [mvanhorn/cli-printing-press](https://github.com/mvanhorn/cli-printing-press) | Generator for agent-first Go CLIs plus MCP servers with local SQLite/FTS sync, compound commands, provenance, and proof gates. |

## Contributing

This is a living document. If you've built an agent-facing CLI and learned something the hard way, PRs are welcome.

## License

MIT
