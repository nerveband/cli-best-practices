# Contract-first design (ShipTypes)

Boris Tane (Cloudflare) published [shiptypes.com](https://shiptypes.com/) with a simple thesis: type definitions should be the primary API contract, not prose documentation.

The argument is that documentation is a lossy copy of the code. It drifts. A type definition can't drift because it IS the code. When an agent has types, it reaches the correct implementation on the first attempt. When it only has prose docs, it needs multiple error-recovery cycles to get there.

Cloudflare's [2026 CLI rebuild](https://blog.cloudflare.com/cf-cli-local-explorer/) shows the same principle applied to a large product surface. Their TypeScript schema is used to generate CLI commands, SDKs, Terraform, MCP, docs, and agent skills, with schema-layer guardrails for naming and flags. The key lesson for smaller CLIs is the same: enforce consistency before review, not after a human notices drift.

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

**Vocabulary rules.** The contract should define canonical verbs and flags and reject banned alternatives in CI. For example: `get`, not `info`; `list`, not only `ls`; one JSON flag; one destructive commitment convention; one pagination vocabulary.

**Local/remote scope.** If a CLI can operate against local simulation and remote production resources, the contract should mark that scope and every response should repeat it. Cloudflare's Local Explorer makes local resources inspectable through an API mirror, which gives agents a safe verification path before touching remote state.

**Skill generation.** `SKILL.md`, `AGENTS.md`, examples, and machine-readable `agent-context` should either be generated from the contract or validated against it. Agent guidance is part of the product surface, not a doc afterthought.

## How I use this

In [agent-to-bricks](https://github.com/nerveband/agent-to-bricks), the CLI has a `bricks schema --validate` command that compares the live CLI behavior against `cli/schema.json`. If anything is out of sync, it fails. This runs in CI on every commit.

In [craft-cli](https://github.com/nerveband/craft-cli), this is on the roadmap. The CLI currently scores 0/3 on schema introspection. Adding a `craft schema` command is the single highest-leverage improvement.
