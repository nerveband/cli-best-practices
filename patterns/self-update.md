# Self-update and version checking

For compiled CLIs distributed as binaries, self-update is table stakes. Agents will use whatever version is installed, and they won't check for updates unless the CLI tells them to.

## Background version check

Check for updates on every run, but don't block execution. Run the check in a goroutine, cache the result, and display a notice if a newer version exists. The user (or agent) can then run `mycli upgrade` when ready.

This is the pattern from [sindresorhus/update-notifier](https://github.com/sindresorhus/update-notifier) (1,795 stars, Node.js) adapted for Go CLIs.

```
$ mycli list
[results...]

A new version of mycli is available: v1.9.0 (current: v1.8.0)
Run 'mycli upgrade' to update.
```

For agents, this message goes to stderr so it doesn't pollute the JSON output on stdout.

## Self-update command

`mycli upgrade` downloads the latest release from GitHub Releases, replaces the current binary, and reports the new version. Libraries that handle this for Go:

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

If your CLI talks to a server (API, plugin, etc.), check version compatibility on every call. Return the server version in a response header (e.g., `X-API-Version`). The CLI can then:

- Warn on minor mismatch: "Server is v2.1, CLI is v2.0. Consider upgrading."
- Hard-block on major mismatch: "Server is v3.0, CLI is v2.x. Upgrade required."

This prevents agents from using a stale CLI against a newer API and getting confusing errors.
