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

## Real-world reference

The [GitHub CLI (`gh`)](https://cli.github.com/) does this well. `gh issue list --json number,title,state` returns only those fields. `gh issue list` without `--json` returns a human-readable table. The non-TTY behavior is automatic.
