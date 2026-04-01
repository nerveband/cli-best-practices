# Agent CLI Audit

A runnable, pass/fail test spec for evaluating CLI agent-friendliness. Designed to be executed by an AI coding agent against any CLI binary.

Unlike the Agent DX Scale (which is a subjective 0-3 rubric), this is a concrete checklist where each item can be verified by running a command and checking the output. An agent can score any CLI in under 5 minutes.

Sourced from: building 10+ CLIs, the Agent DX Scale, Eric Zakariasson's 10 rules, CLIWatch skills protocol, InfoQ agent CLI patterns, GitLab glab #8177, Scalekit MCP benchmarks, ShipTypes contract-first philosophy, AgentDX linter rules, and field experience from craft-cli and agent-to-bricks.

---

## How to run this audit

Replace `$CLI` with the binary name. Run each test. Mark pass/fail. Count the score at the end.

Total: 50 checks across 10 categories. Each check is 1 point.
- 0-15: Human-only
- 16-25: Agent-tolerant
- 26-35: Agent-ready
- 36-45: Agent-first
- 46-50: Best-in-class

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
$CLI schema
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
$CLI <read-command> --format json
# or: $CLI <read-command> --json
# or: $CLI <read-command> (if JSON is default)
```
PASS if: at least one command outputs valid JSON.

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
$CLI <command-that-will-fail> --format json 2>&1
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
# or: $CLI <destructive-command> -y
# or: $CLI <destructive-command> --force
```
PASS if: a flag exists to skip interactive confirmations.

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

## Category 5: Error handling (5 checks)

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

## Category 7: Predictability (4 checks)

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

---

## Category 8: Agent knowledge (5 checks)

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

---

## Category 9: Resilience (4 checks)

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

---

## Category 10: Distribution and lifecycle (3 checks)

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

---

## Scoring

Count your passes. The breakdown by category tells you where to focus:

| Category | Max | Your score |
|----------|-----|------------|
| Discoverability | 7 | |
| Structured output | 6 | |
| Input flexibility | 5 | |
| Safety rails | 6 | |
| Error handling | 5 | |
| Context discipline | 5 | |
| Predictability | 4 | |
| Agent knowledge | 5 | |
| Resilience | 4 | |
| Distribution | 3 | |
| **Total** | **50** | |

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

The audit should take under 5 minutes for a well-structured CLI, 10-15 for a messy one.
