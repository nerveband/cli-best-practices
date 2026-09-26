# Structured output

The single most impactful thing you can do for agent compatibility. If your CLI only prints tables and prose, agents have to parse free text, which is fragile, wastes tokens, and breaks on the next release.

This doc follows [the CLI Spec](https://clispec.dev/) (Principles 1, 3, and 6) and [clig.dev](https://clig.dev/), and adds lessons from real CLIs.

## The format flag

- **One explicit flag, and it always wins.** `--output`/`-o` is canonical. Use `--format` if `-o` already means an output path in your CLI, as it does for curl and most compilers. Keep old spellings such as `--json` working as aliases.
- **Three-valued, with `auto` as the default.** `auto` picks by TTY detection; an explicit `text` or `json` always wins. `mycli -o text list | grep web` must produce text even though it's piped.
- **Structured by default when piped.** New CLIs should emit JSON when stdout isn't a terminal. An existing CLI whose scripts depend on piped text may keep that default, but must declare it in its schema (`"output": {"piped": "text"}`) so agents read the contract instead of guessing.
- **No ANSI when piped.** Respect `NO_COLOR`, and never emit color codes when stdout is not a terminal, even in text mode.

```go
// auto: JSON when piped, table on a terminal. Explicit values always win.
switch format {
case "auto":
    if isatty.IsTerminal(os.Stdout.Fd()) {
        outputTable(result)
    } else {
        outputJSON(result)
    }
case "json":
    outputJSON(result)
default:
    outputTable(result)
}
```

## Streams

Data goes to stdout. Progress, warnings, update notices, and diagnostics go to stderr, in every mode. An agent piping to `jq` should never get `Fetching...` in the JSON, and an agent piping a download to `tar` should never get a progress bar in the archive.

## Envelopes

- **Collections** return `{"items": [...]}`, not a bare array. The envelope gives pagination and truncation metadata a place to live without changing the shape later.
- **Single records** return the document directly.
- **Partial results say so.** Include `total`, `next_cursor`, or `"truncated": true`. An agent that gets 100 silently truncated rows will confidently report that 100 is all there is.
- **Writes return what happened.** Return server-assigned IDs and per-item results, not just "OK". Idempotent writes include `"changed": true|false`, the Terraform and Ansible convention, so the agent knows whether anything actually changed.

```json
{"items": [{"id": "abc", "title": "First doc"}], "total": 1847, "next_cursor": "eyJvIjoxMH0"}
```

## Errors

**Declare a fixed set of error kinds, each with its own exit code.** The exit code is the part that's always machine-readable: it's free to check and survives even when stderr is discarded.

**In JSON mode, write the envelope as the last line of stderr:**

```json
{"error": {"kind": "rate_limit", "message": "Space request budget exhausted",
           "hint": "Retry after 12 seconds, or narrow the query with --limit.",
           "retryable": true,
           "details": {"retry_after_seconds": 12, "scope": "space"}}}
```

- `kind` (required): a stable identifier from the declared set. Agents branch on it.
- `message` (required): human-readable.
- `hint` (optional): what to do next.
- `details` (optional): structured, kind-specific context.

**In text mode, write a normal human error.** CLI Spec v0.3 reversed v0.2 on this. A person at a terminal shouldn't get a line of JSON; the machine contract is the exit code plus the declared kind-to-code mapping, and agents that want the envelope pass the format flag.

**Pass through what the server knows.** When an API returns `Retry-After` or rate-limit headers, put the delay and the budget scope in `details`. Don't collapse a 429 into "retry later". The agent needs the number to back off correctly.

**Enum errors list the valid values.** `invalid --status "done"; valid: todo, active, completed, canceled`.

## Exit codes

There's no universal numbering. What matters is that each error kind has one declared code and the table is documented where agents look (help, schema, `AGENTS.md`). A typical mapping:

| Code | Kind |
|------|------|
| 0 | Success |
| 1 | General or unknown error |
| 2 | Usage or validation error |
| 3 | Auth error |
| 4 | Not found |
| 5 | Conflict |
| 6 | Confirmation required (no TTY, no bypass flag) |
| 7 | Network or upstream API error |
| 8 | Rate limited |

Some non-zero exits aren't errors. `diff` and `grep` exit 1 for "difference found" or "no match". The CLI Spec calls these **outcomes**: declare them separately, write no error envelope, and never reuse an error kind's code for one.

## NDJSON streaming

For large or streaming output, emit one JSON object per line and flush each as it's ready. `head -n 20` of NDJSON is 20 valid records; a truncated JSON array is unparseable.

```bash
$ mycli -o ndjson logs tail
{"ts":"2026-09-26T10:00:01Z","level":"info","msg":"started"}
{"ts":"2026-09-26T10:00:02Z","level":"warn","msg":"slow query"}
```

## Field selection

A `--fields` flag lets agents request only what they need. A full Craft document can be 2,000 tokens; `--fields id,title` makes it 50. The CLI Spec requires field selection on unbounded collections; add it to wide single records too.

```bash
$ mycli -o json docs list --fields id,title
{"items": [{"id": "abc", "title": "First doc"}], "next_cursor": "eyJvIjoxMH0"}
```

## Separate data and error formats

The [OpenAI CLI](https://github.com/openai/openai-cli) exposes independent `--format` and `--format-error` controls, which helps when success and failure take different paths: pretty data for a human, JSON errors for CI. Useful formats beyond `json`: `jsonl`/`ndjson` for streams and batches, `raw` for the exact API payload, and `yaml` for humans editing structured input.

## Built-in transforms

A transform flag handles nested extraction without another shell call to `jq`:

```bash
$ mycli -o json resource list --transform 'items.0.id'
```

The OpenAI CLI uses [GJSON syntax](https://github.com/tidwall/gjson/blob/master/SYNTAX.md). The syntax matters less than making common projections first-class, documented, and consistent for both data and errors.

## Real-world references

The [GitHub CLI](https://cli.github.com/) does this well: `gh issue list --json number,title,state` returns only those fields, and the same command without `--json` prints a table.

[craft-cli](https://github.com/nerveband/craft-cli) defaults to JSON and offers `--format` with `json`, `jsonl`, `yaml`, `raw`, `table`, and `markdown`, plus `--fields`, `--transform`, `--id-only`, and `--json-errors`. Its exit codes are documented in `AGENTS.md`. As of v1.12.0 it still collapses 429 responses into a generic retry message; passing through `Retry-After` is an open fix.

[agent-to-bricks](https://github.com/nerveband/agent-to-bricks) validates its error taxonomy and JSON shapes in CI.
