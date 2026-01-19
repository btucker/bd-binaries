# bd-binaries

Pre-built binaries for [beads (bd)](https://github.com/steveyegge/beads), automatically synced from upstream releases.

## Why This Exists

[Claude Code Web](https://claude.ai/code) runs in a restricted network environment with specific domain allowlists. This creates challenges for installing tools that rely on typical distribution methods:

| Domain | Status | Used For |
|--------|--------|----------|
| `raw.githubusercontent.com` | Allowed | Raw file access, source archives |
| `codeload.github.com` | Allowed | Archive downloads |
| `github.com` | Allowed | Redirects |
| `release-assets.githubusercontent.com` | **Blocked** | GitHub release binaries |
| `storage.googleapis.com` | **Blocked** | Go module proxy |

This means:
- Standard `go install` fails (Go proxy blocked)
- GitHub release binary downloads fail (release assets blocked)
- Building from source fails (can't fetch Go dependencies)

## The Workaround

Since `raw.githubusercontent.com` **is** allowed, we can serve pre-built binaries directly from a Git repository. This repo:

1. Automatically syncs the latest release binaries from [steveyegge/beads](https://github.com/steveyegge/beads)
2. Commits them to this repo, organized by platform
3. Makes them available via `raw.githubusercontent.com`

## Available Binaries

Binaries are organized by platform:

```
linux_amd64/bd    # Linux x86_64
darwin_amd64/bd   # macOS Intel
darwin_arm64/bd   # macOS Apple Silicon
```

## Usage in Claude Code Web

Use these URLs in your session-start hook to download `bd`:

```bash
# Linux (amd64) - most common for Claude Code Web
curl -fsSL https://raw.githubusercontent.com/btucker/bd-binaries/main/linux_amd64/bd -o bd
chmod +x bd
./bd --version

# macOS Intel
curl -fsSL https://raw.githubusercontent.com/btucker/bd-binaries/main/darwin_amd64/bd -o bd

# macOS Apple Silicon
curl -fsSL https://raw.githubusercontent.com/btucker/bd-binaries/main/darwin_arm64/bd -o bd
```

### Example Session-Start Hook

Create a `.claude/hooks/session-start.sh`:

```bash
#!/bin/bash
set -e

BD_PATH="${HOME}/.local/bin/bd"
BD_URL="https://raw.githubusercontent.com/btucker/bd-binaries/main/linux_amd64/bd"

if [ ! -f "$BD_PATH" ]; then
    mkdir -p "$(dirname "$BD_PATH")"
    echo "Installing bd..."
    curl -fsSL "$BD_URL" -o "$BD_PATH"
    chmod +x "$BD_PATH"
    echo "bd installed successfully"
fi

export PATH="${HOME}/.local/bin:$PATH"
```

## Automatic Updates

This repository uses a GitHub Action that:

- Runs daily at 6 AM UTC
- Checks for new releases from `steveyegge/beads`
- Downloads and commits updated binaries automatically
- Can be triggered manually via workflow dispatch

See [`.github/workflows/sync-beads.yml`](.github/workflows/sync-beads.yml) for details.

## Version Tracking

The `VERSION` file in the repo root contains the current synced release tag.

## License

The `bd` binaries are from [steveyegge/beads](https://github.com/steveyegge/beads) - see that repository for licensing information.

This sync infrastructure is provided as-is for the Claude Code Web community.
