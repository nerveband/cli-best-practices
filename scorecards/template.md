# Scorecard: [CLI Name]

**Date:** YYYY-MM-DD
**Version:** vX.Y.Z
**Audit version:** v3 ([agent-cli-audit.md](agent-cli-audit.md))
**Total: ?/97**
**CLI Spec:** conformant to 0.2 / 0.3 candidate / not published / failed (reason)

## Scores

| Category | Score | Notes |
|----------|-------|-------|
| Discoverability | /7 | |
| Structured output | /6 | |
| Input flexibility | /5 | |
| Safety rails | /6 | |
| Error handling | /7 | |
| Context discipline | /5 | |
| Predictability | /7 | |
| Agent knowledge | /7 | |
| Resilience | /7 | |
| Distribution | /5 | |
| Three-layer introspection | /5 | |
| Persistent identity/config | /5 | |
| Two-way I/O/artifacts | /5 | |
| Contract/generation discipline | /5 | |
| Unix composability/restraint | /5 | |
| API-native payload ergonomics | /5 | |
| Domain depth/proof gates | /5 | |
| **Total** | **/97** | |

## Declared exemptions

Checks that passed because the CLI declares why they don't apply.

| Check | Declaration | Where it's declared |
|-------|-------------|---------------------|
| | | |

## Readiness

- [ ] `schema` works with no auth, config, or network
- [ ] Every command declares its effects (read-only, idempotent, non-idempotent)
- [ ] Prompts refuse without a TTY and name the bypass flag
- [ ] Error kinds map to declared exit codes
- [ ] Secrets never required on argv
- [ ] Upstream API contract pinned and diffed in CI (API wrappers only)
- [ ] Installable SKILL.md that follows the Agent Skills spec
- [ ] MCP surface generated from the same contract (if shipped)
- [ ] Async `--wait` and job recovery (if the CLI has async operations)
- [ ] Profiles with inspectable config sources
- [ ] File argument expansion and explicit encodings
- [ ] Local sync/search with explicit data source (high-gravity APIs)
- [ ] Proof-of-behavior or dogfood gates

## Strengths

-

## Gaps

-

## Recommended improvements

1.
2.
3.
