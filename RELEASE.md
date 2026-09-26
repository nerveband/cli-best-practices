# Release Notes

## Audit v3: Standards Alignment (September 2026)

This release aligns the repo with the standards published in 2026 and corrects advice that aged badly. The audit keeps its 85 checks and 17 categories, so earlier scores stay comparable.

### What changed

- **CLI Spec alignment.** Contract-first design, structured output, and safety rails now follow [the CLI Spec](https://clispec.dev/): declared `effects`, `output_kind`, and `cardinality`; error kinds with declared exit codes; human errors in text mode; non-TTY refusal with `confirmation_required`; idempotency keys; `changed` in write results; in-band truncation metadata. v0.2 is frozen and v0.3 is a candidate; the docs say which is which.
- **Declared exemptions.** A check that can't apply to a command passes only when the CLI declares why. Undeclared gaps fail. Flag presence without behavior fails.
- **Safe retries rewritten.** An outcome table replaces "make every command idempotent." Client-side check-then-create no longer counts as idempotency.
- **Trust boundary replaces sanitization.** Remote data is preserved, labeled, and never executed, instead of filtered for known injection patterns. The Agent DX Scale's safety level 3 changes accordingly.
- **Secrets off argv.** Check 3.3 now fails CLIs that only accept secrets as command-line flags.
- **Upstream contract drift.** New guidance and audit criteria (10.3, 14.2) for API wrappers: pin the provider's spec, diff it on a schedule, and test request encoding and response decoding against fixtures.
- **Rate limits.** Check 9.3 requires passing through `Retry-After` and budget scope.
- **Agent Skills spec.** `SKILL.md` guidance and check 8.6 follow the [Agent Skills specification](https://agentskills.io/specification). This repo's own skill is renamed from `cli-best-practices-audit` to `cli-best-practices` to match its directory, as the spec requires. If you installed it under the old directory name, rename the directory.
- **MCP updated.** The CLI vs MCP comparison now reflects the [2026-07-28 MCP specification](https://blog.modelcontextprotocol.io/posts/2026-07-28/) and maps CLI Spec declarations to MCP tool fields and annotations.
- **`AI_AGENT` downgraded.** Agent detection is a proposal, not a standard, and should never change the output contract.
- **Self-update narrowed.** Checksums required, no automatic updates, no checks in CI, and package-manager installs exempt.
- **craft-cli scorecard.** The v1.9.0 scores are kept as history, with open findings from a September 2026 review of v1.12.0 for the next audit.
- **README rewritten** for faster orientation; the full source list moved to the [landscape](ecosystem/landscape-2026.md#sources).

## Agent-Native CLI Checker Upgrade (May 2026)

This release upgrades the CLI checker from a legacy 50-point readiness audit to an 85-point agent-native audit.

### What Changed

- Added [SKILL.md](SKILL.md), a compact agent skill for running the audit consistently.
- Expanded [scorecards/agent-cli-audit.md](scorecards/agent-cli-audit.md) from 10 categories to 17 categories.
- Updated [scorecards/template.md](scorecards/template.md) with the new 85-point reporting table.
- Refreshed README and ecosystem docs with the current agent-native CLI landscape.
- Added pattern guidance for:
  - schema-enforced command vocabulary
  - three-layer introspection
  - async `--wait` and durable job recovery
  - profiles and config precedence
  - artifact delivery and feedback loops
  - API-native payload ergonomics
  - local data layers, compound commands, and proof gates

### What Was Learned

The earlier checker correctly covered the basics: JSON, non-interactive inputs, dry-run, safety rails, useful errors, and bounded output. The newer sources show that best-in-class CLIs now go further.

The upgraded audit now checks whether a CLI compounds agent success over repeated use: whether agents can discover the command surface without wasting context, recover async jobs, reuse profiles, route artifacts, inspect local data, and rely on mechanical proofs rather than hand-wavy docs.

### Sources and Thanks

Thanks to the people and projects that informed this upgrade:

- [Trevin Chow, "10 Principles for Agent-Native CLIs"](https://trevinsays.com/p/10-principles-for-agent-native-clis)
- [Cloudflare, "Building a CLI for all of Cloudflare"](https://blog.cloudflare.com/cf-cli-local-explorer/)
- [heygen-com/heygen-cli](https://github.com/heygen-com/heygen-cli)
- [openai/openai-cli](https://github.com/openai/openai-cli)
- [mvanhorn/cli-printing-press](https://github.com/mvanhorn/cli-printing-press)
- [Hacker News discussion on agent-native CLIs](https://news.ycombinator.com/item?id=48052333)

The HN thread was especially useful as a counterweight: agent-friendly design should not normalize careless destructive flags or abandon Unix composability. The audit now explicitly checks for restraint, human-readable modes, and skills that teach normal CLI composition rather than separate agent-only behavior.

### Compatibility

The original 50-point audit remains available by scoring Categories 1-10 only. New audits should use the full 85-point version.
