# Agent CLI Audit

**By [Ashraf Ali](https://github.com/nerveband)** (original work)

A runnable, pass/fail test spec for evaluating CLI agent-friendliness. Designed to be executed by an AI coding agent against any CLI binary.

Unlike the [Agent DX Scale](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/) (which is a subjective 0-3 rubric), this is a concrete checklist where each item can be verified by running a command and checking the output. An agent can score a small CLI quickly and produce evidence-backed notes for larger CLIs.

Validated against [craft-cli](https://github.com/nerveband/craft-cli) (scored 45/50) and informed by building 10+ CLIs including [agent-to-bricks](https://github.com/nerveband/agent-to-bricks), [beeper-api-cli](https://github.com/nerveband/beeper-api-cli), [yt-api-cli](https://github.com/nerveband/yt-api-cli), [mochi-cli](https://github.com/nerveband/mochi-cli), and [cloak-agent](https://github.com/nerveband/cloak-agent).

This v2 audit keeps the original 50 defensive checks and adds 35 compounding checks from the 2026 agent-native CLI wave: mechanically enforced vocabulary, three-layer introspection, async recovery, profiles, delivery sinks, feedback loops, skill packaging, API-native payload ergonomics, local data layers, compound insight commands, and proof gates.

### Sources that informed this framework

- [Agent DX CLI Scale](https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/) by Justin Poehnelt (the 7-axis rubric this extends into executable tests)
- [Building CLIs for agents](https://x.com/ericzakariasson/status/2036762680401223946) by Eric Zakariasson (10 practical rules)
- [CLI Skills Protocol](https://cliwatch.com/blog/designing-a-cli-skills-protocol) by CLIWatch (skills subcommand pattern)
- [Patterns for AI Agent Driven CLIs](https://www.infoq.com/articles/ai-agent-cli/) by InfoQ (exit codes, non-interactive flags)
- [GitLab CLI agent enhancement #8177](https://gitlab.com/gitlab-org/cli/-/work_items/8177) (--agent-info, --help --format json)
- [MCP vs CLI benchmark](https://www.scalekit.com/blog/mcp-vs-cli-use) by Scalekit (cost and reliability numbers)
- [ShipTypes](https://shiptypes.com/) by Boris Tane (contract-first philosophy)
- [AgentDX linter](https://github.com/agentdx/agentdx) (MCP tool description rules)
- [10 Principles for Agent-Native CLIs](https://trevinsays.com/p/10-principles-for-agent-native-clis) by Trevin Chow (table-stakes vs. compounding model, three-layer introspection, async ledger, profiles, delivery, feedback)
- [Building a CLI for all of Cloudflare](https://blog.cloudflare.com/cf-cli-local-explorer/) by Cloudflare (schema-layer command consistency, local/remote signaling, local explorer API)
- [heygen-com/heygen-cli](https://github.com/heygen-com/heygen-cli) and [HeyGen CLI SKILL.md](https://github.com/heygen-com/heygen-cli/blob/main/SKILL.md) (JSON-first output, request/response schemas, async `--wait`, agent-installable skill)
- [openai/openai-cli](https://github.com/openai/openai-cli) (resource-based command structure, separate data/error output formatting, transforms, file argument expansion, explicit file encodings, debug-log warnings)
- [mvanhorn/cli-printing-press](https://github.com/mvanhorn/cli-printing-press) (local SQLite/FTS sync, compound domain commands, competitor feature absorb gate, provenance manifests, dogfood/proof-of-behavior verification, auth doctor, anti-gaming rules)
- [Hacker News discussion: Principles for agent-native CLIs](https://news.ycombinator.com/item?id=48052333) (counterpressure around `--force`, dry-run-by-default, Unix composability, and skill-as-manpage pattern)

---

## How to run this audit

Replace `$CLI` with the binary name. Run each test. Mark pass/fail. Count the score at the end.

Total: 85 checks across 17 categories. Each check is 1 point.
- 0-20: Human-only
- 21-35: Agent-tolerant
- 36-50: Agent-ready
- 51-65: Agent-first
- 66-85: Agent-native

For legacy comparisons, you can still score Categories 1-10 only as the original 50-point audit.

---

## Category 1: Discoverability (7 checks)

### 1.1 Root help lists subcommands
```bash
$CLI --help
```
PASS if: output lists available subcommands with one-line descriptions. Agent can pick the right subcommand from this output alone.

### 1.2 Every subcommand has --help
```bash
$CLI <subcommand> --help
```
PASS if: every subcommand responds to --help without error.

### 1.3 Help includes examples
```bash
$CLI <subcommand> --help
```
PASS if: at least 80% of subcommands include concrete usage examples (not just flag descriptions).

### 1.4 Examples use realistic values
```bash
$CLI <subcommand> --help
```
PASS if: examples use realistic placeholder values (like actual IDs, dates, names), not generic "VALUE" or "STRING".

### 1.5 Progressive disclosure works
```bash
$CLI
$CLI <group>
$CLI <group> <subcommand> --help
```
PASS if: agent can drill down from root to specific command in 3 steps or fewer, learning only what it needs at each level.

### 1.6 Machine-readable help available
```bash
$CLI agent-context
# or: $CLI schema
# or: $CLI --help --format json
# or: $CLI describe
```
PASS if: some form of machine-readable command manifest exists (JSON output of commands, flags, types).

### 1.7 Version is queryable
```bash
$CLI version
# or: $CLI --version
```
PASS if: outputs version string that can be parsed programmatically.

---

## Category 2: Structured output (6 checks)

### 2.1 JSON output available
```bash
$CLI <read-command> --json
# or: $CLI <read-command> --format json
# or: $CLI <read-command> (if JSON is default)
```
PASS if: at least one command outputs valid JSON. Prefer one canonical JSON convention across the whole CLI, either a dedicated `--json` flag or a consistent global `--format json`.

### 2.2 JSON is consistent across commands
Run 3+ different read commands with JSON output.
PASS if: all produce valid JSON with consistent envelope structure.

### 2.3 JSON is the default when piped
```bash
$CLI <read-command> | cat
```
PASS if: piped output is JSON (or structured), not human-formatted tables.

### 2.4 Errors are structured
```bash
$CLI <command-that-will-fail> --json 2>&1
# or: $CLI <command-that-will-fail> --format-error json 2>&1
# or: $CLI <command-that-will-fail> 2>&1 if errors are always structured
```
PASS if: error output is JSON with at least `code` and `message` fields, not just prose text.

### 2.5 Exit codes are meaningful
```bash
$CLI <invalid-command>; echo $?
$CLI <auth-failure>; echo $?
$CLI <not-found>; echo $?
```
PASS if: different error types produce different non-zero exit codes (not all exit 1).

### 2.6 Quiet mode suppresses noise
```bash
$CLI <command> --quiet
# or: $CLI <command> -q
```
PASS if: a quiet/silent flag exists that suppresses status messages, leaving only data output.

---

## Category 3: Input flexibility (5 checks)

### 3.1 All inputs via flags (non-interactive)
```bash
$CLI <mutating-command> --flag1 value --flag2 value
```
PASS if: every required input can be passed as a flag. No command requires interactive prompts to complete.

### 3.2 Stdin accepted for structured input
```bash
echo '{"key":"value"}' | $CLI <mutating-command> --stdin
# or: $CLI <mutating-command> --json '{"key":"value"}'
```
PASS if: at least mutating commands accept JSON via stdin or --json flag.

### 3.3 Env vars for auth
```bash
export CLI_API_KEY=xxx && $CLI <command>
# or: $CLI <command> --api-key xxx
```
PASS if: credentials can be set via environment variables or flags (no interactive login flow required).

### 3.4 Flags over positional args
```bash
$CLI <command> --help
```
PASS if: most inputs use named flags rather than positional arguments. Positional args limited to 0-1 primary identifier.

### 3.5 Raw payload passthrough
```bash
$CLI <mutating-command> --json '{ full API payload }'
```
PASS if: agent can pass the exact API payload shape without translating to bespoke flags.

---

## Category 4: Safety rails (6 checks)

### 4.1 Dry-run exists
```bash
$CLI <destructive-command> <id> --dry-run
```
PASS if: --dry-run flag exists on at least one mutating command.

### 4.2 Dry-run on ALL mutating commands
Test --dry-run on every command that creates, updates, deletes, or moves.
PASS if: every mutating command supports --dry-run.

### 4.3 Dry-run output describes the action
```bash
$CLI delete <id> --dry-run
```
PASS if: dry-run output clearly states WHAT would happen (not just "dry run mode enabled").

### 4.4 Confirmation skip flag
```bash
$CLI <destructive-command> --yes
# or: $CLI <destructive-command> --commit
# or: $CLI <destructive-command> --force
```
PASS if: a flag exists to skip interactive confirmations or explicitly commit destructive work. Record the chosen convention. HN feedback on the Trevin post objected to training agents to overuse `--force`; `--yes` or `--commit` is often safer unless the CLI's ecosystem has standardized on `--force`.

### 4.5 Idempotent operations
```bash
$CLI <create-or-update> <same-args> # run twice
```
PASS if: running the same command twice doesn't create duplicates or error. Returns "no change" or succeeds silently.

### 4.6 Safety metadata exposed
```bash
$CLI schema
```
PASS if: command metadata includes safety info (readonly, destructive, idempotent, supports_dry_run).

---

## Category 5: Error handling (7 checks)

### 5.1 Errors are actionable
```bash
$CLI <command-missing-required-flag>
```
PASS if: error message tells you exactly what's wrong AND how to fix it (shows correct usage or suggests a flag).

### 5.2 Fail fast on missing input
```bash
$CLI <command> # missing required args
```
PASS if: exits immediately with error, doesn't hang or wait for input.

### 5.3 Network errors are distinct
```bash
$CLI <command> --api-url https://invalid.example.com
```
PASS if: network/API errors have a different exit code than validation errors.

### 5.4 Error includes hint for recovery
```bash
$CLI <auth-failure-command>
```
PASS if: error output includes a "hint" or "suggestion" field pointing to the fix.

### 5.5 Errors to stderr, data to stdout
```bash
$CLI <failing-command> 2>/dev/null  # should show nothing
$CLI <failing-command> 1>/dev/null  # should show the error
```
PASS if: error messages go to stderr, data goes to stdout.

### 5.6 Enum errors enumerate valid values
```bash
$CLI <command> --enum-flag definitely-not-valid
```
PASS if: validation errors for enums or finite choices include the accepted values and the rejected value.

### 5.7 Validation happens before side effects
```bash
$CLI <mutating-command> --bad-flag value --dry-run
```
PASS if: invalid inputs fail before network calls, writes, or destructive side effects.

---

## Category 6: Context window discipline (5 checks)

### 6.1 Field selection available
```bash
$CLI <read-command> --fields id,title
# or: $CLI <read-command> --output-only id
```
PASS if: agent can request only specific fields to reduce token usage.

### 6.2 Pagination or limits
```bash
$CLI <list-command> --limit 5
# or: $CLI <list-command> --max-results 5
```
PASS if: agent can control how many results are returned.

### 6.3 ID-only mode
```bash
$CLI <list-command> --id-only
```
PASS if: a mode exists that returns only identifiers (minimal tokens).

### 6.4 Count without fetching
```bash
$CLI <list-command> --count
```
PASS if: agent can get the count of results without fetching all data.

### 6.5 Depth control
```bash
$CLI <get-command> <id> --max-depth 1
```
PASS if: for nested data, agent can control how deep the response goes.

---

## Category 7: Predictability (7 checks)

### 7.1 Consistent command structure
```bash
$CLI <resource1> list
$CLI <resource2> list
$CLI <resource3> list
```
PASS if: all resources follow the same verb pattern (resource+verb or verb+resource consistently).

### 7.2 Consistent flag names
PASS if: the same concept uses the same flag name across commands (--format, not --format/--output/--type depending on command).

### 7.3 No surprises in output shape
Run the same command 3 times.
PASS if: output structure is identical each time (same keys, same nesting).

### 7.4 Documented exit code table
```bash
$CLI --help
# or: check AGENTS.md / README
```
PASS if: exit codes are documented somewhere the agent can find them.

### 7.5 Canonical verbs match common agent expectations
```bash
$CLI <resource> list
$CLI <resource> get <id>
$CLI <resource> create --help
$CLI <resource> update --help
$CLI <resource> delete --help
```
PASS if: common resource operations use predictable verbs such as `list`, `get`, `create`, `update`, and `delete`, not surprising aliases like `info` for `get` or `ls` as the only listing verb.

### 7.6 Canonical flags are enforced across commands
PASS if: the CLI uses one spelling for major cross-cutting flags, especially the JSON format convention (`--json` or global `--format json`), `--limit`, `--dry-run`, `--yes`/`--commit`, `--timeout`, and `--profile`.

### 7.7 Banned aliases are checked mechanically
PASS if: CI, codegen, schema linting, or tests fail when banned verbs/flags are introduced. Manual review alone does not pass.

---

## Category 8: Agent knowledge (7 checks)

### 8.1 AGENTS.md exists
```bash
cat AGENTS.md
```
PASS if: repo root contains AGENTS.md with usage guidance.

### 8.2 AGENTS.md includes guardrails
PASS if: AGENTS.md contains explicit agent instructions like "always use --dry-run before delete" or "use --fields to limit output."

### 8.3 Workflow examples exist
PASS if: documentation includes multi-step workflow examples (not just single-command reference).

### 8.4 Common mistakes documented
PASS if: documentation warns about pitfalls or common agent errors with this CLI.

### 8.5 Prompts or skills shipped
```bash
ls prompts/ skills/ SKILL.md 2>/dev/null
```
PASS if: repo ships session prompts (implement/check/release) or SKILL.md files for AI coding agents.

### 8.6 SKILL.md is installable and scoped
```bash
cat SKILL.md
```
PASS if: the skill has valid frontmatter, a concise trigger description, auth/setup notes, safe default workflows, and explicit "do not" guidance for agent misuse.

### 8.7 Skill and docs are validated against the live CLI
PASS if: docs, skills, and examples are generated from the same schema as the CLI or checked in CI by running examples/help/schema against the built binary.

---

## Category 9: Resilience (7 checks)

### 9.1 Timeout handling
```bash
$CLI <command> --timeout 1  # or test with slow network
```
PASS if: CLI handles timeouts gracefully with a clear error, not a stack trace.

### 9.2 Partial failure reporting
```bash
$CLI <batch-command> # with some valid, some invalid items
```
PASS if: batch operations report which items succeeded and which failed, not just "error."

### 9.3 Retry guidance
PASS if: transient error messages indicate whether retrying is appropriate.

### 9.4 Graceful degradation
```bash
$CLI <command> # with expired auth
```
PASS if: auth failures produce a clear message with re-auth instructions, not a cryptic 401.

### 9.5 Async commands support --wait
```bash
$CLI <async-create-command> --wait --timeout 30s
```
PASS if: async submission commands can block until completion with bounded timeout and machine-readable final or partial output.

### 9.6 Durable job ledger exists
```bash
$CLI jobs list
$CLI jobs get <job-id>
```
PASS if: async jobs are recorded across invocations so a killed or retried agent can resume without submitting a duplicate job.

### 9.7 Async retry policy is documented
PASS if: polling backoff, retryable status codes, timeout exit code, and resume command are documented in help, agent-context, or SKILL.md.

---

## Category 10: Distribution and lifecycle (5 checks)

### 10.1 Single binary or simple install
```bash
which $CLI
```
PASS if: CLI is a single binary or installable with one command (brew, go install, curl | sh). No runtime dependencies.

### 10.2 Self-update
```bash
$CLI upgrade
# or: $CLI update
```
PASS if: CLI can update itself without external tooling.

### 10.3 Version mismatch warning
PASS if: CLI warns when it detects a version mismatch with the API/server it talks to.

### 10.4 Agent-friendly auth setup
```bash
$CLI auth status
echo "$TOKEN" | $CLI auth login
```
PASS if: auth can be configured non-interactively, status can be checked without mutation, and credential precedence is documented.

### 10.5 Offline/local mode is explicit when available
```bash
$CLI <command> --local --json
# or: $CLI local <resource> list --json
```
PASS if: local vs. remote execution is explicit in flags and output metadata whenever the CLI can talk to both local simulation and remote production resources.

---

## Category 11: Three-layer introspection (5 checks)

### 11.1 Human help exists
```bash
$CLI --help
$CLI <group> --help
```
PASS if: help is concise, progressive, and usable by both humans and agents.

### 11.2 Agent-context is versioned
```bash
$CLI agent-context --json
# or: $CLI schema --json
```
PASS if: the machine-readable manifest includes a schema/version field so consumers can detect breaking changes.

### 11.3 Agent-context exposes command metadata
PASS if: the manifest includes commands, flags, input types, required fields, output shape, examples, exit codes, and safety metadata.

### 11.4 Request/response schemas are available
```bash
$CLI <command> --request-schema
$CLI <command> --response-schema
```
PASS if: data-bearing commands can emit JSON Schema or equivalent contract data without auth or network calls.

### 11.5 Skill path is discoverable
```bash
$CLI skill-path
# or: $CLI skills
```
PASS if: the CLI can point agents to bundled `SKILL.md` files or emit task-oriented skill metadata.

---

## Category 12: Persistent identity and configuration (5 checks)

### 12.1 Profiles are supported
```bash
$CLI profile list
$CLI profile show <name>
```
PASS if: recurring configuration can be saved as named profiles rather than repeated in every invocation.

### 12.2 Profile precedence is documented
PASS if: precedence is clear, ideally explicit flag > environment variable > profile > default.

### 12.3 Profiles are exposed through agent-context
PASS if: available profile names and safe non-secret profile metadata appear in the machine-readable manifest.

### 12.4 Config source can be inspected
```bash
$CLI config list --json
```
PASS if: settings can be listed with their source (default, profile, config file, env, flag) without leaking secrets.

### 12.5 Secrets are separated from non-secret config
PASS if: credentials are stored and reported separately from normal config, with redaction in all structured output.

---

## Category 13: Two-way I/O and artifacts (5 checks)

### 13.1 Artifact delivery sinks exist
```bash
$CLI <artifact-command> --deliver stdout
$CLI <artifact-command> --deliver file:./out.ext
```
PASS if: artifact-producing commands can route output to stdout and files without forcing agents to scrape prose or move temporary files.

### 13.2 Delivery writes are atomic
PASS if: file delivery writes atomically or refuses to overwrite unless an explicit overwrite/commit flag is present.

### 13.3 Unknown delivery schemes enumerate supported values
```bash
$CLI <artifact-command> --deliver definitely-unknown:target
```
PASS if: the error names supported schemes such as `stdout`, `file:<path>`, and `webhook:<url>`.

### 13.4 Feedback can be recorded locally
```bash
$CLI feedback "flag X rejected value Y but docs imply it should work"
```
PASS if: agents can record friction locally in a structured log without needing network access.

### 13.5 Optional upstream feedback is discoverable
PASS if: an upstream feedback endpoint, when configured, is surfaced in agent-context without exposing secrets.

---

## Category 14: Contract and generation discipline (5 checks)

### 14.1 One source of truth exists
PASS if: command definitions, docs, skills, API wrappers, SDK/MCP wrappers, and examples derive from or validate against one schema/contract.

### 14.2 Contract validation runs in CI
```bash
$CLI schema --validate
# or: npm test / go test includes schema drift checks
```
PASS if: CI fails when CLI behavior, docs, or generated artifacts drift from the contract.

### 14.3 Generated files are clearly marked
PASS if: generated command surfaces live in clearly marked paths and contributors are instructed not to hand-edit them.

### 14.4 Local/remote scope is part of the contract
PASS if: the schema declares whether a command affects local state, remote state, or both, and output repeats that scope.

### 14.5 Tool/MCP descriptions are token-budgeted
PASS if: any MCP or tool-wrapper surface is generated with concise descriptions and a checked budget, not large prose copied from docs.

---

## Category 15: Unix composability and agent restraint (5 checks)

### 15.1 Human-readable mode remains available
```bash
$CLI <read-command> --human
# or: $CLI <read-command> --table
```
PASS if: agent-first output does not remove useful human/programmatic CLI behavior.

### 15.2 JSON is concise, not a dump
PASS if: default JSON returns the useful envelope and bounded data, not every nested API field by default.

### 15.3 Dangerous work requires explicit commitment
PASS if: destructive commands either default to preview/non-commit or require an explicit confirmation-bypass/commit flag in non-interactive contexts.

### 15.4 Flag aliases improve usability without hiding canonical names
PASS if: common aliases are accepted only when they do not undermine safety or consistency, and help/schema identify the canonical flag.

### 15.5 Skill guidance favors composition over special agent-only behavior
PASS if: skills teach agents how to compose normal CLI commands instead of requiring a separate, incompatible agent behavior layer.

---

## Category 16: API-native payload ergonomics (5 checks)

### 16.1 Resource-based command structure maps to the API
```bash
$CLI <resource> <command> --help
```
PASS if: the CLI command hierarchy mirrors the API/resource model closely enough that agents can transfer API knowledge to CLI usage.

### 16.2 Data and error output formats are independently configurable
```bash
$CLI <read-command> --format json
$CLI <failing-command> --format-error json
```
PASS if: agents can choose machine-readable output for success and error streams independently.

### 16.3 Multiple structured output formats exist when useful
```bash
$CLI <read-command> --format jsonl
$CLI <read-command> --format raw
$CLI <read-command> --format yaml
```
PASS if: the CLI offers formats that fit different automation contexts, especially `json`, `jsonl`, and `raw` for streaming or exact API responses.

### 16.4 Output transforms are built in
```bash
$CLI <read-command> --transform 'data.0.id'
```
PASS if: agents can extract a field or projection without piping large responses through a second tool.

### 16.5 File arguments are first-class and explicit
```bash
$CLI <command> --arg @file.txt
$CLI <command> --arg @file://file.txt
$CLI <command> --arg @data://image.png
```
PASS if: files can be passed directly, inside structured payloads when needed, escaped when a literal starts with `@`, and explicitly encoded as text or base64/binary.

---

## Category 17: Domain depth and proof gates (5 checks)

### 17.1 Local data layer exists for high-gravity resources
```bash
$CLI sync --help
$CLI search --help
```
PASS if: APIs with large, frequently queried, or historical resources provide local persistence such as SQLite, FTS/search, and incremental sync instead of forcing every agent query through live API calls.

### 17.2 Data source is explicit and controllable
```bash
$CLI <query-command> --data-source local
$CLI <query-command> --data-source live
$CLI <query-command> --data-source auto
```
PASS if: agents can choose local, live, or bounded auto-refresh behavior and the response says which source was used.

### 17.3 Compound domain commands go beyond endpoint mirroring
```bash
$CLI health --help
$CLI stale --help
$CLI reconcile --help
```
PASS if: the CLI includes domain-shaped workflow or insight commands that answer common multi-resource questions in one call.

### 17.4 Proof-of-behavior checks exist
```bash
$CLI doctor --json
# or: make dogfood / make verify / <repo-local proof command>
```
PASS if: verification proves path validity, flag wiring, auth format, data pipeline read/write paths, and absence of dead generated code.

### 17.5 Provenance and competitor coverage are recorded
PASS if: generated or curated CLIs include a provenance manifest, source/inspiration citations, and a competitor feature checklist so the CLI is judged against real user workflows, not just its own scorecard.

---

## Scoring

Count your passes. The breakdown by category tells you where to focus:

| Category | Max | Your score |
|----------|-----|------------|
| Discoverability | 7 | |
| Structured output | 6 | |
| Input flexibility | 5 | |
| Safety rails | 6 | |
| Error handling | 7 | |
| Context discipline | 5 | |
| Predictability | 7 | |
| Agent knowledge | 7 | |
| Resilience | 7 | |
| Distribution | 5 | |
| Three-layer introspection | 5 | |
| Persistent identity/config | 5 | |
| Two-way I/O/artifacts | 5 | |
| Contract/generation discipline | 5 | |
| Unix composability/restraint | 5 | |
| API-native payload ergonomics | 5 | |
| Domain depth/proof gates | 5 | |
| **Total** | **85** | |

---

## Running this as an AI agent

If you're an AI coding agent asked to audit a CLI, do this:

1. Build the binary: `go build` / `npm run build` / etc.
2. Run `$CLI --help` to get the command list
3. Work through each category, running the actual commands
4. For mutating commands, use `--dry-run` to test safely
5. Record pass/fail for each check with a one-line note
6. Output the scorecard with category breakdowns
7. List the top 5 highest-impact fixes (biggest point gains with least effort)
8. Cite the sources that justify non-obvious recommendations, especially when proposing `--force` vs. `--yes`/`--commit`, schema/codegen enforcement, async ledgers, profiles, delivery, feedback, or skill packaging

The legacy 50-point audit should take under 5 minutes for a well-structured CLI. The full 85-point audit usually takes 10-25 minutes, depending on how many async, profile, artifact, contract, API-payload, local-store, and proof-gate checks apply.
