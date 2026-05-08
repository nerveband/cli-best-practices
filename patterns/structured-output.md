# Structured output

The single most impactful thing you can do for agent compatibility. If your CLI only outputs human-readable tables and prose, agents have to parse free text, which is fragile, token-wasteful, and error-prone.

## The basics

**JSON is the default in piped contexts.** When stdout isn't a terminal (i.e., the output is being piped to another command or captured by an agent), switch to JSON automatically. When it IS a terminal, human-friendly output is fine.

```go
// Detect TTY
if !isatty.IsTerminal(os.Stdout.Fd()) {
    outputJSON(result)
} else {
    outputTable(result)
}
```

**Errors are JSON too.** This is the part most CLIs miss. They'll return JSON for success but print `Error: something went wrong` as plain text to stderr. An agent needs structured errors just as much as structured success.

```json
{
  "error": {
    "code": "AUTH_FAILED",
    "message": "API key is invalid or expired",
    "hint": "Run: craft config add <name> <url>"
  }
}
```

**Exit codes are meaningful.** Don't just use 0 and 1. Define a taxonomy:

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General / unknown error |
| 2 | API / network error |
| 3 | Config error |
| 4 | Validation error |
| 5 | Conflict (resource already exists, version mismatch) |

Agents can branch on exit codes without parsing the error message.

## Advanced: NDJSON streaming

For commands that return paginated data, emit one JSON object per line (Newline-Delimited JSON). The agent can process results as they arrive without buffering the entire response.

```bash
$ mycli list --format ndjson
{"id":"abc","title":"First doc"}
{"id":"def","title":"Second doc"}
{"id":"ghi","title":"Third doc"}
```

## The `--fields` flag

Agents don't need every field in the response. A `--fields` flag lets them request only what they need, saving tokens.

```bash
$ mycli list --fields id,title
[{"id":"abc","title":"First doc"},{"id":"def","title":"Second doc"}]
```

This matters more than you'd think. A full Craft document response might be 2,000 tokens. With `--fields id,title`, it's 50.

## Separate data and error formatting

Some CLIs need richer formatting than a single `--json` switch. The [OpenAI CLI](https://github.com/openai/openai-cli) exposes independent data and error format controls:

```bash
$ openai responses create --format json --format-error json
```

That separation is useful for agents because success and failure often take different paths. A human might want pretty output for data while keeping errors machine-readable in CI, or an agent might request raw data while using structured JSON errors for recovery.

Useful formats:

- `json` for normal structured automation.
- `jsonl` for streams or batches.
- `raw` when the exact API payload matters.
- `yaml` for humans editing structured inputs or outputs.

## Built-in transforms

Field selection is the simple case. A transform flag handles nested extraction without forcing the agent to spend another shell call on `jq` or a custom script:

```bash
$ mycli resource list --format json --transform 'data.0.id'
```

The OpenAI CLI uses [GJSON syntax](https://github.com/tidwall/gjson/blob/master/SYNTAX.md) for `--transform` and `--transform-error`. The broader best practice is not the specific syntax; it is making common projection operations first-class, documented, and consistent for both data and error streams.

## Real-world references

The [GitHub CLI (`gh`)](https://cli.github.com/) does this well. `gh issue list --json number,title,state` returns only those fields. `gh issue list` without `--json` returns a human-readable table. The non-TTY behavior is automatic.

In [craft-cli](https://github.com/nerveband/craft-cli), JSON is the default output format. `--output-only <field>` and `--id-only` reduce output to just the fields an agent needs. `--json-errors` returns structured errors with error codes, messages, and hints. Exit codes are categorized: 0 success, 1 user error, 2 API error, 3 config error.

In [agent-to-bricks](https://github.com/nerveband/agent-to-bricks), the error system uses a formal taxonomy (CONFIG exit 2, API exit 3, Validation exit 4, Conflict exit 5) with structured JSON shapes validated in CI.
