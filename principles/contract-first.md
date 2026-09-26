# Contract-first design

Boris Tane (Cloudflare) published [shiptypes.com](https://shiptypes.com/) with a simple thesis: type definitions should be the primary API contract, not prose documentation. Documentation is a lossy copy of the code and it drifts. When an agent has types, it reaches the correct call on the first attempt. When it only has prose, it needs several error-recovery cycles.

Cloudflare's [2026 CLI rebuild](https://blog.cloudflare.com/cf-cli-local-explorer/) applies this at scale. One TypeScript schema generates CLI commands, SDKs, Terraform, MCP, docs, and agent skills, with schema-layer guardrails for naming and flags. The lesson for smaller CLIs is the same: enforce consistency before review, not after someone notices drift.

A CLI has two contracts to keep honest: the one it **publishes** to agents, and the one it **consumes** from an upstream API.

## The published contract

**One canonical source.** A schema defines every command, argument, output shape, and error kind. Help text, docs, skills, and any MCP surface are generated from it or validated against it.

**Use a published format.** [The CLI Spec](https://clispec.dev/) defines a JSON schema document for exactly this, and a checker for it. [v0.2](https://clispec.dev/spec/v0.2/) is frozen and safe to claim conformance to. v0.3 is a candidate (August 2026) that you can build against, knowing it may still change. Richer formats such as [OpenCLI](https://opencli.org/) or [usage](https://usage.jdx.dev/) are good for docs and shell completions; the CLI Spec schema is the agent-facing layer on top.

### Declare what each command is

v0.3's key idea: every command describes itself, and only the rules that fit that description apply. The declaration is what earns an exemption.

| Declaration | Values | What it tells the consumer |
|---|---|---|
| `effects` (required) | `read_only`, `idempotent`, `non_idempotent` | Whether the command changes anything, and whether repeating it is safe |
| `output_kind` | `data` (default), `stream`, `opaque` | Whether stdout is one JSON document, a line-by-line stream, or raw bytes such as a file |
| `cardinality` (data only) | `single`, `bounded`, `unbounded` | Whether pagination and field selection are required |
| `errors` | kinds from the top-level list | Which failures this command can produce, each with a declared exit code |
| `confirmation_bypass_arg` | e.g. `--yes` | The command prompts on a TTY and refuses without one |
| `idempotency_key_arg` | e.g. `--request-id` | How to make a non-idempotent command safe to retry |

```json
{
  "clispec": "0.3",
  "name": "mycli",
  "version": "2.3.0",
  "output": {"tty": "text", "piped": "json"},
  "commands": [
    {"name": "docs list", "description": "List documents.",
     "effects": "read_only", "cardinality": "unbounded",
     "pagination": {"style": "cursor", "cursor_field": "next_cursor",
                    "cursor_arg": "--cursor", "limit_arg": "--limit"},
     "fields_arg": "--fields",
     "output_fields": [{"name": "id", "type": "string"},
                       {"name": "title", "type": "string"}],
     "errors": ["auth", "rate_limit"]},
    {"name": "docs delete", "description": "Move a document to trash.",
     "effects": "idempotent", "cardinality": "single",
     "confirmation_bypass_arg": "--yes",
     "errors": ["auth", "not_found", "confirmation_required"]}
  ],
  "errors": [
    {"kind": "auth", "exit_code": 3, "retryable": false},
    {"kind": "rate_limit", "exit_code": 8, "retryable": true}
  ]
}
```

(Abbreviated: a real document also declares each command's `args` and every error kind it references.)

**Declare, never infer.** Safety metadata derived from a command's name (`delete` means destructive, `get` means read-only) is wrong often enough to be dangerous. A nested `views delete` and a top-level `folders delete` can have very different blast radii, and a `get` that rotates a token is not read-only. Permission systems auto-approve on `read_only`, and retry loops trust `idempotent`, so an inferred claim is a liability. Write the declaration next to the command definition and test it.

### The schema command's own contract

- It works **before anything else does**: no auth, no config file, no network. Agents reach for it exactly when they know nothing, often after setup failed.
- Root `--help` mentions it, because `--help` is the universal first probe.
- It accepts a command path to narrow the output (`mycli schema docs list`), because a full dump for a large CLI wastes context.

### CI fails on drift

If you add a flag and don't update the schema, the build breaks. If help text, `SKILL.md`, or an MCP tool description disagrees with the schema, the build breaks. In [agent-to-bricks](https://github.com/nerveband/agent-to-bricks), `bricks schema --validate` compares live CLI behavior against `cli/schema.json` on every commit.

## The consumed contract

A CLI that wraps an API inherits that API's contract. When the provider changes a parameter name or adds a response field, the CLI can break silently: requests still succeed, but filters are ignored or data is dropped.

- **Pin a snapshot.** Check in the provider's OpenAPI or docs export with its source URL, fetch date, and a content hash.
- **Diff it on a schedule.** A scheduled CI job refetches the spec and fails on any change, so drift becomes a reviewed diff instead of a user bug report.
- **Test encoding and decoding against fixtures.** Assert exact query parameter names, casing, and array encoding. Decode every documented response variant. Keep default tests offline; use opt-in read-only probes only when a fixture can't settle a question.
- **Don't flatten responses.** Model discriminated unions properly, or keep unknown fields as raw JSON, so new server fields pass through instead of vanishing.
- **Keep write results.** When the API returns per-item results or server-assigned IDs, return them. Discarding them hides partial failures.

A September 2026 static audit of craft-cli found the client sending `folderIDs` where the current Craft docs specify `folderIds`, a mismatch that could silently drop a search filter. The same audit found REST endpoints the CLI still labeled MCP-only. Neither shows up in unit tests that only mock the CLI's own assumptions. A pinned contract plus request-encoding fixtures catches both.

## Other contract rules

**Vocabulary.** Define canonical verbs and flags and reject banned alternatives in CI: `get`, not `info`; `list`, not only `ls`; one format flag; one confirmation convention; one pagination vocabulary.

**Local and remote scope.** If a CLI can operate on a local simulation and on remote production resources, the contract marks which one each command touches and every response repeats it. Cloudflare's Local Explorer gives agents an inspectable local mirror, which is a safe place to verify before touching remote state.

**Generated agent guidance.** `SKILL.md`, `AGENTS.md`, examples, and MCP tool definitions are part of the product surface. Generate them from the contract or validate them against it.

## How I use this

[agent-to-bricks](https://github.com/nerveband/agent-to-bricks) validates its schema in CI, as described above.

[craft-cli](https://github.com/nerveband/craft-cli) ships `craft schema` with flag types and safety metadata. As of v1.12.0 that metadata is inferred from leaf command names (`inferSafety` in `cmd/schema.go`), and the pinned Craft REST contract in `docs/contracts/` is behind the live API. Declared per-command effects and a scheduled contract diff are the next steps.
