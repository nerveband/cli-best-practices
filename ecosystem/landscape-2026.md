# The agent-friendly CLI landscape in 2026

A snapshot of where things stand as of May 2026. The MCP vs CLI debate dominated dev discourse in Q1. The dust is settling into "use both." Meanwhile, a handful of standards are emerging for how CLIs should present themselves to agents.

## The MCP vs CLI debate

It started with Eric Holmes's ["MCP is dead. Long live the CLI"](https://ejholmes.github.io/2026/02/28/mcp-is-dead-long-live-the-cli.html) (~400 HN points) and spawned dozens of response posts.

The numbers from Scalekit's [75-run benchmark](https://www.scalekit.com/blog/mcp-vs-cli-use):

| Metric | CLI | MCP |
|--------|-----|-----|
| Cost per interaction | 1x | 10-32x |
| Failure rate | 0% | 28% |
| Schema tokens | ~0 (model already knows `gh`) | ~55,000 (GitHub MCP server) |
| Monthly cost at 10K/day | baseline | +$500-$2K |

The reason CLIs are cheaper: LLMs were trained on millions of man pages, READMEs, and shell scripts. MCP launched in late 2024 with nearly zero training data. Every MCP interaction depends on schemas loaded at runtime.

The consensus, from [CircleCI's synthesis](https://circleci.com/blog/mcp-vs-cli/): CLIs for the inner loop (developer speed, token efficiency). MCP for the outer loop (multi-tenant auth, enterprise governance). Most production setups will use both.

## Emerging standards

### Agent runtime detection

[`@vercel/detect-agent`](https://www.npmjs.com/package/@vercel/detect-agent) checks for the `AI_AGENT` environment variable, then falls back to 9+ tool-specific vars (Cursor, Claude Code, Devin, etc.). The [AGENTS.md proposal (Issue #136)](https://github.com/agentsmd/agents.md/issues/136) wants this standardized like `CI=true`.

What to do now: check for `AI_AGENT` env var and auto-switch to JSON output, structured errors, no progress bars.

### CLI Skills Protocol

CLIWatch proposed a [`skills` subcommand](https://cliwatch.com/blog/designing-a-cli-skills-protocol) that returns structured JSON describing workflows. Their testing showed GPT-5.2 went from 33% to 50% pass rate with it, and token usage dropped from 13K to 8K.

Lark's CLI ([larksuite/cli](https://github.com/larksuite/cli)) ships 19 built-in agent skills across 200+ commands. It's the best reference implementation I've seen.

### SKILL.md format

Gaining adoption across Claude Code, Cursor, Gemini CLI, and others. YAML frontmatter with triggers, followed by workflow steps in markdown. Teams shipping SKILL.md files alongside their CLIs report much higher agent success rates. [GitBook's explainer](https://www.gitbook.com/blog/skill-md) is a good starting point.

### Agent-native compounding layer

Trevin Chow's [10 Principles for Agent-Native CLIs](https://trevinsays.com/p/10-principles-for-agent-native-clis) reframed the field as two layers. The table stakes are non-interactive commands, structured output, teaching errors, safe retries, and bounded responses. The compounding layer is newer: cross-CLI vocabulary consistency, three-layer introspection, async-aware execution, persistent profiles, and two-way I/O for artifact delivery plus feedback.

### Schema-enforced vocabulary

Cloudflare's [CLI for all of Cloudflare](https://blog.cloudflare.com/cf-cli-local-explorer/) is the clearest large-scale implementation. Their TypeScript schema generates multiple interfaces and enforces conventions like `get` over `info`, a single JSON flag, and consistent confirmation bypass vocabulary. It also highlights a subtle agent problem: local and remote resources must be explicit in both defaults and output, or an agent can mutate one environment while inspecting another.

### Safety counterpressure

The [HN discussion](https://news.ycombinator.com/item?id=48052333) around agent-native CLIs is worth reading because it argues against cargo-culting every agent-specific pattern. Useful takeaways: avoid training agents to casually use `--force`; consider dry-run-by-default with explicit `--commit`; preserve Unix composability and human-readable modes; and treat `SKILL.md` as a concise manpage for agents rather than a separate incompatible agent-only interface.

### API-native payload ergonomics

The official [OpenAI CLI](https://github.com/openai/openai-cli) adds several practical patterns for CLIs that wrap large REST APIs: resource-based commands, env var plus flag credentials, separate `--format` and `--format-error`, JSON/JSONL/raw/YAML modes, GJSON-style output transforms, `@file` expansion inside arguments and structured blobs, explicit `@file://` vs. `@data://` encodings, and prominent warnings that debug logs can contain sensitive payloads.

These are not replacements for the agent-native compounding layer. They are API-wrapper ergonomics that make the lower-level command surface easier for agents and shell scripts to compose without extra glue code.

### Local-first domain CLIs

[CLI Printing Press](https://github.com/mvanhorn/cli-printing-press) argues that the best agent CLIs should not stop at endpoint mirroring. Its generated CLIs add SQLite persistence, FTS search, incremental sync, `--data-source local|live|auto`, compound domain commands, provenance manifests, competitor feature absorption, and mechanical verification gates.

The important checker lesson is anti-gaming: a CLI should not pass merely because it has `--json`, `--dry-run`, and nice help text. For high-gravity APIs, audit whether the CLI can answer the questions agents actually ask in one bounded command, whether the local data pipeline is wired end to end, and whether proof-of-behavior checks catch hallucinated paths, dead flags, auth mismatches, and unread tables.

### GitLab's blueprint

[GitLab CLI Issue #8177](https://gitlab.com/gitlab-org/cli/-/work_items/8177) lays out a practical plan: `--agent-info` flag, `--help --format json` for machine-readable help, consistent exit codes, `glab doctor` for self-diagnosis, structured error JSON. Worth studying if you're planning similar work.

## Repos worth knowing about

### Tools

| Repo | What |
|------|------|
| [HKUDS/CLI-Anything](https://github.com/HKUDS/CLI-Anything) | Auto-generates CLIs from source code for agents. 13K+ stars in a week. |
| [agentdx/agentdx](https://github.com/agentdx/agentdx) | "ESLint for MCP servers." 30-rule linter, 0-100 score. |
| [brwse/earl](https://github.com/brwse/earl) | AI-safe CLI. Operations are HCL templates, not free-form commands. Blocks SSRF. |
| [larksuite/cli](https://github.com/larksuite/cli) | 200+ commands, 19 agent skills, MIT. Reference implementation. |

### Agent-friendly CLI examples

| Repo | What |
|------|------|
| [nerveband/craft-cli](https://github.com/nerveband/craft-cli) | Craft.do documents. 30+ commands, schema introspection, 45/50 audit score. |
| [nerveband/agent-to-bricks](https://github.com/nerveband/agent-to-bricks) | Bricks Builder bridge. 56 commands, contract-first, CI-validated schema. |
| [nerveband/beeper-api-cli](https://github.com/nerveband/beeper-api-cli) | Cross-platform messaging CLI (WhatsApp, Telegram, Signal). |
| [nerveband/yt-api-cli](https://github.com/nerveband/yt-api-cli) | YouTube Data API v3 CLI. |
| [nerveband/mochi-cli](https://github.com/nerveband/mochi-cli) | Mochi.cards flashcard management. Built for LLM automation. |
| [nerveband/cloak-agent](https://github.com/nerveband/cloak-agent) | Stealth browser automation for AI agents. |
| [heygen-com/heygen-cli](https://github.com/heygen-com/heygen-cli) | Official HeyGen video API CLI. JSON stdout, structured stderr, request/response schemas, non-interactive auth, async `--wait`, and bundled skill guidance. |
| [openai/openai-cli](https://github.com/openai/openai-cli) | Official OpenAI REST API CLI. Resource-based command tree, data/error format controls, transforms, and explicit file argument encodings. |
| [mvanhorn/cli-printing-press](https://github.com/mvanhorn/cli-printing-press) | Agent-first CLI/MCP generator with SQLite sync, offline search, compound domain commands, provenance, and proof gates. |
| [jaredpalmer/mogcli](https://github.com/jaredpalmer/mogcli) | Agent-friendly M365 CLI. By Jared Palmer (Vercel). |
| [dl-alexandre/Google-Play-Developer-CLI](https://github.com/dl-alexandre/Google-Play-Developer-CLI) | Agent-friendly Play Console CLI. |
| [zcaceres/builtwith-api](https://github.com/zcaceres/builtwith-api) | Shows the CLI + MCP dual-interface pattern. |
| [Gladium-AI/n8n-cli](https://github.com/Gladium-AI/n8n-cli) | Agent-friendly n8n workflow automation. |

### Reference and curation

| Repo | What |
|------|------|
| [cli-guidelines/cli-guidelines](https://github.com/cli-guidelines/cli-guidelines) | The canonical CLI design guide. 3,573 stars. [clig.dev](https://clig.dev/) |
| [bradAGI/awesome-cli-coding-agents](https://github.com/bradAGI/awesome-cli-coding-agents) | Directory of terminal-native AI coding agents. |
| [agarrharr/awesome-cli-apps](https://github.com/agarrharr/awesome-cli-apps) | 19K+ stars. Catalog of well-designed CLIs to study. |
| [clifor.ai](https://www.clifor.ai/) | Curated directory of CLI tools for agents, with reviews. |

## Articles and posts

| Title | Author/Source | Link |
|-------|---------------|------|
| Patterns for AI Agent Driven CLIs | InfoQ | [link](https://www.infoq.com/articles/ai-agent-cli/) |
| Designing a CLI Skills Protocol | CLIWatch | [link](https://cliwatch.com/blog/designing-a-cli-skills-protocol) |
| Building CLIs for agents (10 rules) | Eric Zakariasson | [link](https://x.com/ericzakariasson/status/2036762680401223946) |
| Rewrite Your CLI for AI Agents | Justin Poehnelt | [link](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/) |
| 10 Principles for Agent-Native CLIs | Trevin Chow | [link](https://trevinsays.com/p/10-principles-for-agent-native-clis) |
| Building a CLI for all of Cloudflare | Cloudflare | [link](https://blog.cloudflare.com/cf-cli-local-explorer/) |
| Principles for agent-native CLIs | Hacker News | [link](https://news.ycombinator.com/item?id=48052333) |
| OpenAI CLI | OpenAI | [link](https://github.com/openai/openai-cli) |
| CLI Printing Press | Michael VanHorn | [link](https://github.com/mvanhorn/cli-printing-press) |
| Ship Types, Not Docs | Boris Tane | [link](https://shiptypes.com/) |
| Writing CLI Tools Agents Want to Use | Uche Enyioha | [link](https://dev.to/uenyioha/writing-cli-tools-that-ai-agents-actually-want-to-use-39no) |
| 10 Principles for Delightful CLIs | Atlassian | [link](https://www.atlassian.com/blog/it-teams/10-design-principles-for-delightful-clis) |
| CLI Design Guidelines | Thoughtworks | [link](https://www.thoughtworks.com/insights/blog/engineering-effectiveness/elevate-developer-experiences-cli-design-guidelines) |
| CLI UX: Progress Display Patterns | Evil Martians | [link](https://evilmartians.com/chronicles/cli-ux-best-practices-3-patterns-for-improving-progress-displays) |
| Your MCP Server Is Eating Your Context Window | Apideck | [link](https://www.apideck.com/blog/mcp-server-eating-context-window-cli-alternative) |
| BetterCLI.org | BetterCLI | [link](https://bettercli.org/) |
| skill.md explained | GitBook | [link](https://www.gitbook.com/blog/skill-md) |
