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

### Recommended Session-Start Hook

Create a `.claude/hooks/session-start.sh` that tries multiple installation methods with this repo as the fallback:

```bash
#!/bin/bash

# .claude/hooks/session-start.sh
echo "Setting up bd (beads issue tracker)..."

# Try npm first, fall back to go install, then binary download
if ! command -v bd &> /dev/null; then
    if npm install -g @beads/bd --quiet 2>/dev/null && command -v bd &> /dev/null; then
        echo "Installed via npm"
    elif command -v go &> /dev/null && go install github.com/steveyegge/beads/cmd/bd@latest 2>/dev/null; then
        export PATH="$PATH:$HOME/go/bin"
        echo "Installed via go install"
    else
        # Fallback: download pre-built binary (works in Claude Code Web)
        echo "Trying binary download fallback..."
        BD_PATH="${HOME}/.local/bin/bd"
        mkdir -p "$(dirname "$BD_PATH")"
        curl -fsSL https://raw.githubusercontent.com/btucker/bd-binaries/main/linux_amd64/bd -o "$BD_PATH"
        chmod +x "$BD_PATH"
        export PATH="${HOME}/.local/bin:$PATH"
        echo "Installed via binary download"
    fi
fi

# Verify and show version
bd version
```

This hook:
1. Tries `npm install` first (fastest if available)
2. Falls back to `go install` (if Go is installed)
3. Falls back to downloading the pre-built binary from this repo (works in Claude Code Web where npm/go are blocked)

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
