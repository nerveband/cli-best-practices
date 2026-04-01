# The agent-friendly CLI landscape in 2026

A snapshot of where things stand as of April 2026. The MCP vs CLI debate dominated dev discourse in Q1. The dust is settling into "use both." Meanwhile, a handful of standards are emerging for how CLIs should present themselves to agents.

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
| Ship Types, Not Docs | Boris Tane | [link](https://shiptypes.com/) |
| Writing CLI Tools Agents Want to Use | Uche Enyioha | [link](https://dev.to/uenyioha/writing-cli-tools-that-ai-agents-actually-want-to-use-39no) |
| 10 Principles for Delightful CLIs | Atlassian | [link](https://www.atlassian.com/blog/it-teams/10-design-principles-for-delightful-clis) |
| CLI Design Guidelines | Thoughtworks | [link](https://www.thoughtworks.com/insights/blog/engineering-effectiveness/elevate-developer-experiences-cli-design-guidelines) |
| CLI UX: Progress Display Patterns | Evil Martians | [link](https://evilmartians.com/chronicles/cli-ux-best-practices-3-patterns-for-improving-progress-displays) |
| Your MCP Server Is Eating Your Context Window | Apideck | [link](https://www.apideck.com/blog/mcp-server-eating-context-window-cli-alternative) |
| BetterCLI.org | BetterCLI | [link](https://bettercli.org/) |
| skill.md explained | GitBook | [link](https://www.gitbook.com/blog/skill-md) |
