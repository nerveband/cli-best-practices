# CLI Best Practices for AI Agents

Rules, patterns, and a runnable audit for building command-line tools that AI agents can use safely, and that humans and scripts still enjoy.

Agents are not people at a keyboard. They can't answer "are you sure?" prompts, they guess flags that don't exist, they retry after timeouts, and every line of output costs them context. A CLI that handles this well gets better for everyone who uses it.

This repo collects what I've learned building 10+ agent-facing CLIs, checked against current standards and the wider community. By [Ashraf Ali](https://github.com/nerveband).

## Start here

| If you want to... | Read |
|---|---|
| Learn the essentials in five minutes | [The core rules](#the-core-rules) below |
| Audit a CLI, by hand or with an agent | [SKILL.md](SKILL.md), then the [Agent CLI Audit](scorecards/agent-cli-audit.md) |
| Design the machine-readable contract | [Contract-first design](principles/contract-first.md) |
| Get output, errors, and exit codes right | [Structured output](patterns/structured-output.md) |
| Make writes and retries safe | [Safety rails](patterns/safety-rails.md) |
| Teach agents how to use your CLI | [Agent knowledge packaging](patterns/agent-knowledge.md) |
| See the standards, tools, and sources | [The 2026 landscape](ecosystem/landscape-2026.md) |

## The core rules

1. **Structured output on request, and by default when piped.** Offer an explicit format flag (`--output json` or `--format json`). Data goes to stdout; progress and diagnostics go to stderr. No color codes when stdout is not a terminal.
2. **Never hang without a terminal.** Every input works as a flag, environment variable, file, or stdin. A command that would prompt must instead refuse with an error that names the bypass flag, such as `--yes`. It must never proceed silently.
3. **Errors that teach.** Declare a fixed set of error kinds, each with its own exit code. In JSON mode, emit `{"error": {"kind", "message", "hint"}}`. List the valid values when an enum is wrong, and pass through retry timing when the server provides it.
4. **Say what re-running does.** Declare every command as read-only, idempotent, or non-idempotent. Idempotent commands report `"changed": false` when nothing changed. Non-idempotent commands accept an idempotency key or document how to check the outcome.
5. **Preview before committing.** Support `--dry-run` on destructive and non-idempotent commands, and say whether the preview was checked by the server or only computed locally.
6. **Bounded output.** Paginate collections that can grow without limit, support field selection, and say in the output when a result is partial (`total`, `next_cursor`, or `truncated: true`).
7. **A schema command that works first.** `mycli schema` describes commands, arguments, output shapes, and error kinds with no auth, config, or network. Validate it in CI.
8. **Keep the upstream contract honest.** If your CLI wraps an API, pin a copy of the API's spec, diff it on a schedule, and test request encoding and response decoding against fixtures.
9. **Treat remote data as data.** Preserve it faithfully, label where it came from, and never follow instructions found inside it.
10. **Keep secrets out of argv.** Read credentials from environment variables, stdin, a keychain, or a saved profile. Never require `--token abc123` on the command line.
11. **Ship a short skill.** A spec-compliant `SKILL.md` that teaches workflows and guardrails, backed by the schema, beats long docs.
12. **Stay a good Unix citizen.** Keep a human-readable mode, consistent verbs and flags, and composable output. Agent features should improve the normal CLI, not replace it with an agent-only one.

Rules 1 to 7 follow [the CLI Spec](https://clispec.dev/) and [clig.dev](https://clig.dev/). The rest, and the async, profile, artifact, and local-data checks in the audit, go further based on experience with real CLIs.

## Score a CLI

| Tool | What it measures | Use it when |
|---|---|---|
| [Agent CLI Audit](scorecards/agent-cli-audit.md) (original) | 97 pass/fail checks in 17 categories, each verified by running a command | You want an evidence-backed score and a prioritized fix list |
| [CLI Spec conformance](https://clispec.dev/#conformance) | Whether your `schema` output and runtime behavior meet the published spec | You want an external, versioned standard |
| [Agent DX Scale](principles/agent-dx-scale.md) | 7 axes scored 0 to 3, adapted from Justin Poehnelt | You want a quick, subjective read |

A check that doesn't fit a command (pagination on a single record, `--wait` with no async jobs) passes only when the CLI declares why. Undeclared means fail. See [scoring rules](scorecards/agent-cli-audit.md#how-to-run-this-audit).

Scorecards: [ahd-figma](scorecards/ahd-figma.md), [craft-cli](scorecards/craft-cli.md), and a blank [template](scorecards/template.md).

## Use it as an agent skill

[SKILL.md](SKILL.md) follows the [Agent Skills specification](https://agentskills.io/specification). Install it in any harness that loads skills, or point your agent at it before an audit. It tells the agent to:

1. Build or locate the binary.
2. Read root help, the version, and `schema`, then run one safe read command.
3. Work through the audit, using dry-run, fixtures, or mocked credentials for anything that writes.
4. Return a category table with evidence and the five highest-impact fixes.

## CLI and MCP

Use both, generated from one contract. A CLI is cheap for agents that already know shell conventions, easy to compose, and easy for humans to debug. MCP adds typed tool schemas, structured results, per-request auth, and central governance.

The early-2026 benchmarks that found CLIs 10 to 32 times cheaper were driven largely by MCP servers that sent tens of thousands of tokens of tool definitions per session. The [2026-07-28 MCP specification](https://blog.modelcontextprotocol.io/posts/2026-07-28/) adds cacheable tool lists and a stateless core, so treat those numbers as a warning about bloated tool surfaces, not a permanent verdict. More in [the landscape](ecosystem/landscape-2026.md#cli-and-mcp).

## Repository map

| Path | Contents |
|---|---|
| [principles/](principles/) | Why the patterns matter: [contract-first design](principles/contract-first.md), the [Agent DX Scale](principles/agent-dx-scale.md), and [ten rules](principles/ten-rules.md) from Eric Zakariasson |
| [patterns/](patterns/) | How to build them: [structured output](patterns/structured-output.md), [safety rails](patterns/safety-rails.md), [agent knowledge](patterns/agent-knowledge.md), [self-update](patterns/self-update.md) |
| [scorecards/](scorecards/) | The audit, a template, and completed scorecards |
| [ecosystem/](ecosystem/) | Standards, tools, articles, and the full source list |
| [SKILL.md](SKILL.md) | Agent entry point for running an audit |
| [RELEASE.md](RELEASE.md) | What changed in each version |

## CLIs built with these patterns

Every pattern here was tested on these tools.

| CLI | Language | What it does | Last scored |
|---|---|---|---|
| [craft-cli](https://github.com/nerveband/craft-cli) | Go | Craft.do documents, blocks, tasks, collections, and whiteboards over REST and MCP | 83/97 audit v3 on an unreleased build (baseline 45/97), 14/21 DX (v1.9.0). [Scorecard](scorecards/craft-cli.md). |
| [ai-happy-design](https://github.com/nerveband/ai-happy-design) | Go | Figma CLI for AI agents: 144 commands, schema validation, design intelligence | 50/50 legacy audit, 21/21 DX (v0.12.0, April 2026) |
| [agent-to-bricks](https://github.com/nerveband/agent-to-bricks) | Go | Bricks Builder bridge for AI agents, contract-first with CI-validated schema | 9/21 DX |
| [beeper-api-cli](https://github.com/nerveband/beeper-api-cli) | Go | Cross-platform messaging (WhatsApp, Telegram, Signal, and more) | |
| [yt-api-cli](https://github.com/nerveband/yt-api-cli) | Go | YouTube Data API v3: videos, playlists, uploads | |
| [mochi-cli](https://github.com/nerveband/mochi-cli) | Go | Mochi.cards flashcard management, built for LLM automation | |
| [cloak-agent](https://github.com/nerveband/cloak-agent) | Go, TS | Stealth browser automation for AI agents | |
| [zpick](https://github.com/nerveband/zpick) | Go | Single-keypress zmosh session launcher | |
| [drafts-applescript-cli](https://github.com/nerveband/drafts-applescript-cli) | Go | Drafts app CLI (fork) | |
| [image_sense](https://github.com/nerveband/image_sense) | Python | AI image processing with EXIF metadata writing | |
| [token-vision](https://github.com/nerveband/token-vision) | Python | Offline image token calculator for AI models | |

## Credits

The original work here is the [Agent CLI Audit](scorecards/agent-cli-audit.md), the scorecards, and the synthesis in each pattern doc. It builds on [the CLI Spec](https://clispec.dev/) by Ruben Jongejan, [clig.dev](https://clig.dev/), Justin Poehnelt's [Agent DX Scale](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/), Trevin Chow's [agent-native principles](https://trevinsays.com/p/10-principles-for-agent-native-clis), Cloudflare's [CLI rebuild](https://blog.cloudflare.com/cf-cli-local-explorer/), and many others. Every doc cites its sources inline, and the [landscape](ecosystem/landscape-2026.md#sources) has the full list.

## Contributing

This is a living document. If you've built an agent-facing CLI and learned something the hard way, pull requests are welcome.

## License

MIT
