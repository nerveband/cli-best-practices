# Contract-first design (ShipTypes)

Boris Tane (Cloudflare) published [shiptypes.com](https://shiptypes.com/) with a simple thesis: type definitions should be the primary API contract, not prose documentation.

The argument is that documentation is a lossy copy of the code. It drifts. A type definition can't drift because it IS the code. When an agent has types, it reaches the correct implementation on the first attempt. When it only has prose docs, it needs multiple error-recovery cycles to get there.

## In practice

For CLIs, this means:

**One canonical contract.** A `schema.json` or OpenAPI spec that defines every command, flag, payload shape, and error format. Everything else (help text, docs, skill files) is generated from or validated against this contract.

**CI fails on drift.** If you add a flag to the CLI but don't update the schema, the build breaks. If the API changes a response shape but the CLI still expects the old one, the build breaks.

**Safety metadata per action.** Each command declares whether it's:
- `readonly` or mutating
- `destructive` or safe
- `idempotent` or not
- Whether it supports `--dry-run`
- Whether it requires confirmation

An agent can read this metadata and decide how much caution to exercise without being told by a human.

**Machine-readable capability discovery.** Instead of scraping `--help` text (which is designed for human eyes), the agent calls `mycli schema` and gets a structured JSON manifest of everything the CLI can do.

## How I use this

In [agent-to-bricks](https://github.com/nerveband/agent-to-bricks), the CLI has a `bricks schema --validate` command that compares the live CLI behavior against `cli/schema.json`. If anything is out of sync, it fails. This runs in CI on every commit.

In [craft-cli](https://github.com/nerveband/craft-cli), this is on the roadmap. The CLI currently scores 0/3 on schema introspection. Adding a `craft schema` command is the single highest-leverage improvement.
