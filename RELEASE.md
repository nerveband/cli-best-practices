# Release Notes

## Agent-Native CLI Checker Upgrade

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
