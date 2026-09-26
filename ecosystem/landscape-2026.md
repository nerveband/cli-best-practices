# The agent-friendly CLI landscape in 2026

A snapshot as of September 2026. The first half of the year was dominated by the MCP vs CLI debate. The second half brought published standards: the CLI Spec for agent-facing command contracts, the Agent Skills specification for skill packaging, and a major MCP revision.

## Standards

### The CLI Spec

[The CLI Spec](https://clispec.dev/) by Ruben Jongejan defines six principles (structured output, schema introspection, stdout/stderr separation, non-interactive by default, safe retries, bounded output) and a JSON schema for a CLI's `schema` command, with a checker (`clispec score`).

- **v0.2 is frozen.** Claim conformance to it.
- **v0.3 is a candidate (August 2026).** Each command declares its `effects`, `output_kind`, and `cardinality`, and only the rules that fit apply. It also fixed a v0.2 mistake: in text mode, errors should be human-readable, with the machine contract carried by declared exit codes.

This repo's audit aligns with v0.3. See [contract-first design](../principles/contract-first.md).

### Agent Skills

The [Agent Skills specification](https://agentskills.io/specification) standardizes `SKILL.md`: required `name` and `description` frontmatter, optional `license`, `compatibility`, `metadata`, and `allowed-tools`, progressive disclosure (metadata at startup, body on activation, referenced files on demand), and a validator (`skills-ref validate`). See [agent knowledge packaging](../patterns/agent-knowledge.md#skillmd).

### MCP 2026-07-28

The [2026-07-28 MCP specification](https://blog.modelcontextprotocol.io/posts/2026-07-28/) moved to a stateless request/response core, added cacheable and deterministically ordered tool lists, Multi Round-Trip Requests for mid-call input, header-based routing, and authorization hardening. Tasks became an extension. Roots, Sampling, and Logging are deprecated with a twelve-month window.

For CLI authors, the relevant parts are in the [tools specification](https://modelcontextprotocol.io/specification/2026-07-28/server/tools): `inputSchema`, `outputSchema` with `structuredContent`, `isError` for tool failures, and behavior annotations (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`) that clients must treat as untrusted unless the server is trusted.

### clig.dev

[Command Line Interface Guidelines](https://clig.dev/) remains the general reference for CLI conventions. Agent-facing guidance builds on it rather than replacing it.

## CLI and MCP

The debate started with Eric Holmes's ["MCP is dead. Long live the CLI"](https://ejholmes.github.io/2026/02/28/mcp-is-dead-long-live-the-cli.html) and spawned dozens of responses. The most cited numbers came from Scalekit's [75-run benchmark](https://www.scalekit.com/blog/mcp-vs-cli-use):

| Metric | CLI | MCP |
|--------|-----|-----|
| Cost per interaction | 1x | 10-32x |
| Failure rate | 0% | 28% (connection timeouts) |
| Schema tokens | about 0 (model already knows `gh`) | about 55,000 (GitHub MCP server) |
| Monthly cost at 10K interactions/day | baseline | +$500 to $2K |

Read these carefully. The cost gap was driven mostly by one large server sending its full tool list every session, and the failures were transport timeouts. The 2026-07-28 spec's cacheable tool lists and stateless core target exactly those problems. The durable lessons: keep tool surfaces small and descriptions budgeted, and don't make an agent load what it won't use.

The consensus from [CircleCI's synthesis](https://circleci.com/blog/mcp-vs-cli/) still holds: CLIs for the inner loop (developer speed, composability, token efficiency), MCP for the outer loop (multi-tenant auth, governance, typed results). Most production setups use both, ideally generated from one contract.

## Patterns and counterpoints

### Agent-native compounding layer

Trevin Chow's [10 Principles for Agent-Native CLIs](https://trevinsays.com/p/10-principles-for-agent-native-clis) split the field into two layers. Table stakes: non-interactive commands, structured output, teaching errors, safe retries, bounded responses. The compounding layer: consistent vocabulary, three-layer introspection, async-aware execution, persistent profiles, and two-way I/O for artifacts and feedback.

### Schema-enforced vocabulary

Cloudflare's [CLI for all of Cloudflare](https://blog.cloudflare.com/cf-cli-local-explorer/) generates multiple interfaces from one TypeScript schema and enforces conventions such as `get` over `info`, a single JSON flag, and one confirmation vocabulary. It also highlights a subtle risk: local and remote resources must be explicit in both defaults and output, or an agent can mutate one environment while inspecting another.

### Safety counterpressure

The [Hacker News discussion](https://news.ycombinator.com/item?id=48052333) of agent-native CLIs argues against cargo-culting agent-specific patterns: don't train agents to reach for `--force`, consider dry-run by default with explicit `--commit`, keep Unix composability and human-readable modes, and treat `SKILL.md` as a concise manpage rather than a separate agent-only interface.

### API-native payload ergonomics

The [OpenAI CLI](https://github.com/openai/openai-cli) shows practical patterns for wrapping large REST APIs: resource-based commands, env var plus flag credentials, separate `--format` and `--format-error`, JSON/JSONL/raw/YAML modes, GJSON-style transforms, `@file` expansion, explicit `@file://` and `@data://` encodings, and warnings that debug logs can contain sensitive payloads.

### Upstream contract drift

CLIs that wrap hosted APIs break quietly when the provider changes: a renamed query parameter is ignored, a new response field disappears in a typed model. A September 2026 review of craft-cli found both kinds. Pin the provider's spec, diff it on a schedule, and test request encoding against fixtures. See [the consumed contract](../principles/contract-first.md#the-consumed-contract).

### Local-first domain CLIs

[CLI Printing Press](https://github.com/mvanhorn/cli-printing-press) argues that the best agent CLIs shouldn't stop at endpoint mirroring. Its generated CLIs add SQLite persistence, full-text search, incremental sync, `--data-source local|live|auto`, compound domain commands, provenance manifests, competitor feature coverage, and mechanical verification gates.

The checker lesson is anti-gaming: a CLI shouldn't pass merely because it has `--json`, `--dry-run`, and nice help. Audit whether it answers the questions agents actually ask in one bounded command, and whether proof-of-behavior checks catch dead flags, hallucinated paths, and auth mismatches.

### Skills commands

CLIWatch proposed a [`skills` subcommand](https://cliwatch.com/blog/designing-a-cli-skills-protocol) that returns structured JSON describing workflows; their testing showed a GPT-5.2 pass rate rising from 33% to 50% and token use dropping from 13K to 8K. Lark's CLI ([larksuite/cli](https://github.com/larksuite/cli)) ships 19 built-in agent skills across 200+ commands. The CLI Spec `schema` command now covers the machine-readable half of this; `SKILL.md` covers workflows.

### GitLab's blueprint

[GitLab CLI issue #8177](https://gitlab.com/gitlab-org/cli/-/work_items/8177) lays out a practical plan: an `--agent-info` flag, `--help --format json`, consistent exit codes, `glab doctor` for self-diagnosis, and structured error JSON.

### Agent runtime detection

[`@vercel/detect-agent`](https://www.npmjs.com/package/@vercel/detect-agent) checks an `AI_AGENT` environment variable, then falls back to tool-specific variables. An [AGENTS.md proposal](https://github.com/agentsmd/agents.md/issues/136) would standardize it like `CI=true`. It remains a proposal. Use it only for cosmetic choices; letting it change the output contract makes agent and script runs behave differently. See [agent detection](../patterns/agent-knowledge.md#agent-detection).

## Repos worth knowing

### Tools

| Repo | What |
|------|------|
| [rvben/clispec](https://github.com/rvben/clispec) | The CLI Spec source, schema, and `make check` validator |
| [rvben/clispec-cli](https://github.com/rvben/clispec-cli) | `clispec score`, which scores a binary against the runtime checklist |
| [agentskills/agentskills](https://github.com/agentskills/agentskills) | Agent Skills specification and the `skills-ref` validator |
| [HKUDS/CLI-Anything](https://github.com/HKUDS/CLI-Anything) | Auto-generates CLIs from source code for agents |
| [agentdx/agentdx](https://github.com/agentdx/agentdx) | "ESLint for MCP servers": 30-rule linter, 0-100 score |
| [brwse/earl](https://github.com/brwse/earl) | AI-safe CLI where operations are HCL templates, not free-form commands |
| [larksuite/cli](https://github.com/larksuite/cli) | 200+ commands and 19 agent skills; a reference implementation |

### Agent-friendly CLI examples

| Repo | What |
|------|------|
| [nerveband/craft-cli](https://github.com/nerveband/craft-cli) | Craft.do documents over REST and MCP, schema introspection |
| [nerveband/agent-to-bricks](https://github.com/nerveband/agent-to-bricks) | Bricks Builder bridge, contract-first with CI-validated schema |
| [nerveband/beeper-api-cli](https://github.com/nerveband/beeper-api-cli) | Cross-platform messaging CLI (WhatsApp, Telegram, Signal) |
| [nerveband/yt-api-cli](https://github.com/nerveband/yt-api-cli) | YouTube Data API v3 CLI |
| [nerveband/mochi-cli](https://github.com/nerveband/mochi-cli) | Mochi.cards flashcard management, built for LLM automation |
| [nerveband/cloak-agent](https://github.com/nerveband/cloak-agent) | Stealth browser automation for AI agents |
| [heygen-com/heygen-cli](https://github.com/heygen-com/heygen-cli) | JSON stdout, structured stderr, request/response schemas, non-interactive auth, async `--wait`, bundled skill |
| [openai/openai-cli](https://github.com/openai/openai-cli) | Resource-based command tree, data/error format controls, transforms, explicit file encodings |
| [mvanhorn/cli-printing-press](https://github.com/mvanhorn/cli-printing-press) | Agent-first CLI/MCP generator with SQLite sync, offline search, compound commands, provenance, proof gates |
| [jaredpalmer/mogcli](https://github.com/jaredpalmer/mogcli) | Agent-friendly Microsoft 365 CLI |
| [dl-alexandre/Google-Play-Developer-CLI](https://github.com/dl-alexandre/Google-Play-Developer-CLI) | Agent-friendly Play Console CLI |
| [zcaceres/builtwith-api](https://github.com/zcaceres/builtwith-api) | The CLI plus MCP dual-interface pattern |
| [Gladium-AI/n8n-cli](https://github.com/Gladium-AI/n8n-cli) | Agent-friendly n8n workflow automation |

The CLI Spec also lists [reference implementations](https://clispec.dev/#reference-implementations), including clihatch, which scaffolds new spec-compliant tools.

### Reference and curation

| Repo | What |
|------|------|
| [cli-guidelines/cli-guidelines](https://github.com/cli-guidelines/cli-guidelines) | The source of [clig.dev](https://clig.dev/) |
| [bradAGI/awesome-cli-coding-agents](https://github.com/bradAGI/awesome-cli-coding-agents) | Directory of terminal-native AI coding agents |
| [agarrharr/awesome-cli-apps](https://github.com/agarrharr/awesome-cli-apps) | Catalog of well-designed CLIs to study |
| [clifor.ai](https://www.clifor.ai/) | Curated directory of CLI tools for agents |
| [@vercel/detect-agent](https://www.npmjs.com/package/@vercel/detect-agent) | AI agent runtime detection |

## Sources

Every doc in this repo cites its sources inline. This is the full list.

### Specifications

| Title | Author | Link |
|-------|--------|------|
| The CLI Spec | Ruben Jongejan | [clispec.dev](https://clispec.dev/) |
| Agent Skills specification | Agent Skills | [agentskills.io](https://agentskills.io/specification) |
| MCP specification 2026-07-28 | Model Context Protocol | [release notes](https://blog.modelcontextprotocol.io/posts/2026-07-28/), [tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
| Command Line Interface Guidelines | clig.dev | [clig.dev](https://clig.dev/) |

### Articles and posts

| Title | Author | Link |
|-------|--------|------|
| Rewrite Your CLI for AI Agents | Justin Poehnelt | [link](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/) |
| 10 Principles for Agent-Native CLIs | Trevin Chow | [link](https://trevinsays.com/p/10-principles-for-agent-native-clis) |
| Building a CLI for all of Cloudflare | Matt Taylor, Dimitri Mitropoulos, Dan Carter | [link](https://blog.cloudflare.com/cf-cli-local-explorer/) |
| Building CLIs for agents (10 rules) | Eric Zakariasson (Cursor) | [link](https://x.com/ericzakariasson/status/2036762680401223946) |
| Ship Types, Not Docs | Boris Tane | [link](https://shiptypes.com/) |
| Patterns for AI Agent Driven CLIs | InfoQ | [link](https://www.infoq.com/articles/ai-agent-cli/) |
| Designing a CLI Skills Protocol | CLIWatch | [link](https://cliwatch.com/blog/designing-a-cli-skills-protocol) |
| MCP is dead. Long live the CLI | Eric Holmes | [link](https://ejholmes.github.io/2026/02/28/mcp-is-dead-long-live-the-cli.html) |
| MCP vs CLI: Benchmarking Cost and Reliability | Scalekit | [link](https://www.scalekit.com/blog/mcp-vs-cli-use) |
| MCP vs CLI for AI-native development | CircleCI | [link](https://circleci.com/blog/mcp-vs-cli/) |
| Your MCP Server Is Eating Your Context Window | Apideck | [link](https://www.apideck.com/blog/mcp-server-eating-context-window-cli-alternative) |
| One CLI, Two Audiences | Checkly | [link](https://www.checklyhq.com/blog/agentic-cli/) |
| Writing CLI Tools Agents Want to Use | Uche Enyioha | [link](https://dev.to/uenyioha/writing-cli-tools-that-ai-agents-actually-want-to-use-39no) |
| 10 Design Principles for Delightful CLIs | Atlassian | [link](https://www.atlassian.com/blog/it-teams/10-design-principles-for-delightful-clis) |
| CLI Design Guidelines | Thoughtworks | [link](https://www.thoughtworks.com/insights/blog/engineering-effectiveness/elevate-developer-experiences-cli-design-guidelines) |
| CLI UX: Progress Display Patterns | Evil Martians | [link](https://evilmartians.com/chronicles/cli-ux-best-practices-3-patterns-for-improving-progress-displays) |
| skill.md explained | GitBook | [link](https://www.gitbook.com/blog/skill-md) |
| BetterCLI.org | BetterCLI | [link](https://bettercli.org/) |
| GitLab CLI agent enhancement #8177 | GitLab | [link](https://gitlab.com/gitlab-org/cli/-/work_items/8177) |
| Principles for agent-native CLIs | Hacker News | [link](https://news.ycombinator.com/item?id=48052333) |
