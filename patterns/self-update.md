# Self-update and version checking

For compiled CLIs distributed as standalone binaries, a self-update path saves agents from running stale versions against a newer API. It's not universal: if your CLI ships through Homebrew, npm, or a system package manager, defer to that manager instead of replacing the binary behind its back.

## Background version check

Check for updates without blocking execution. Run the check concurrently, cache the result (once a day is plenty), and print a notice if a newer version exists. The user or agent runs `mycli upgrade` when ready.

This is the pattern from [sindresorhus/update-notifier](https://github.com/sindresorhus/update-notifier), adapted for Go CLIs.

```
$ mycli list
[results...]

A new version of mycli is available: v1.9.0 (current: v1.8.0)
Run 'mycli upgrade' to update.
```

Rules that keep it safe for automation:

- **The notice goes to stderr**, never stdout, so it can't corrupt JSON output.
- **Skip it in CI and non-interactive runs** (when `CI` is set or stderr isn't a terminal), and honor an opt-out environment variable. A network call on every invocation slows agents and leaks usage to your update server.
- **Never update automatically.** Replacing the binary mid-task changes behavior under an agent that already read the old schema.

## Self-update command

`mycli upgrade` downloads the latest release, **verifies its checksum or signature**, replaces the current binary atomically, and reports the old and new versions in structured output. Support `--dry-run` to show what would be installed. Libraries that handle this for Go:

| Library | Stars | Notes |
|---------|-------|-------|
| [sanbornm/go-selfupdate](https://github.com/sanbornm/go-selfupdate) | 1,684 | Most popular, mature |
| [minio/selfupdate](https://github.com/minio/selfupdate) | 906 | From MinIO team, handles checksums and signatures |
| [rhysd/go-github-selfupdate](https://github.com/rhysd/go-github-selfupdate) | 641 | Purpose-built for GitHub Releases, detects platform/arch |
| [creativeprojects/go-selfupdate](https://github.com/creativeprojects/go-selfupdate) | 131 | More recent, actively maintained |

For a Go CLI using goreleaser and GitHub Releases, `rhysd/go-github-selfupdate` is the most direct fit.

## goreleaser

Most Go CLIs use [goreleaser](https://goreleaser.com/) for release automation. The config handles:

- Multi-platform builds (Linux, macOS, Windows)
- Multi-architecture (amd64, arm64)
- Checksum generation
- Changelog from commit messages
- GitHub Release creation with assets

A minimal `.goreleaser.yml`:

```yaml
builds:
  - main: .
    binary: mycli
    goos: [linux, darwin, windows]
    goarch: [amd64, arm64]

archives:
  - format_overrides:
      - goos: windows
        format: zip

checksum:
  name_template: 'checksums.txt'

changelog:
  filters:
    exclude:
      - '^docs:'
      - '^test:'
      - '^ci:'
```

## Version mismatch handling

If your CLI talks to a server, check compatibility. Read the server or API version from a response header (for example `X-API-Version`) or a version endpoint. The CLI can then:

- Warn on a minor mismatch: "Server is v2.1, CLI is v2.0. Consider upgrading."
- Refuse on a major mismatch: "Server is v3.0, CLI is v2.x. Upgrade required."

Many hosted APIs don't expose a version at all. For those, the equivalent protection is a pinned copy of the API contract that CI diffs against the live one on a schedule. See [contract-first design](../principles/contract-first.md#the-consumed-contract).
