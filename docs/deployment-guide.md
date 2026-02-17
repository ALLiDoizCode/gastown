# Gas Town - Deployment Guide

**Generated:** 2026-02-17

## Overview

Gas Town is distributed as pre-compiled binaries for multiple platforms via:
- GitHub Releases (direct download)
- Homebrew (macOS/Linux)
- NPM (Node.js ecosystem)

This guide covers deployment, installation, and distribution processes.

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Source Repository                         │
│              github.com/steveyegge/gastown                   │
└─────────────────────────────────────────────────────────────┘
                          │
                    git tag v*.*.*
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              GitHub Actions (.github/workflows)              │
│  ┌──────────┐  ┌──────────┐  ┌────────────┐  ┌──────────┐  │
│  │CI        │  │E2E Tests │  │Integration │  │Release   │  │
│  │Pipeline  │  │          │  │Tests       │  │Workflow  │  │
│  └──────────┘  └──────────┘  └────────────┘  └──────────┘  │
└─────────────────────────────────────────────────────────────┘
                          │
                    GoReleaser v2
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│GitHub Release│  │   Homebrew   │  │     NPM      │
│   Binaries   │  │   Formula    │  │   Package    │
└──────────────┘  └──────────────┘  └──────────────┘
        │                 │                 │
        ▼                 ▼                 ▼
   End Users        macOS/Linux         Node.js Users
```

## Platform Support

### Supported Platforms

| Platform | Architecture | CGO | Distribution Method |
|----------|--------------|-----|---------------------|
| **Linux** | amd64 | ✓ | GitHub, Homebrew, NPM |
| **Linux** | arm64 | ✓ | GitHub, Homebrew, NPM |
| **macOS** | amd64 (Intel) | ✓ | GitHub, Homebrew, NPM |
| **macOS** | arm64 (Apple Silicon) | ✓ | GitHub, Homebrew, NPM |
| **Windows** | amd64 | ✓ | GitHub, NPM |
| **FreeBSD** | amd64 | ✗ | GitHub |

### Platform Notes
- **CGO Enabled:** Required for SQLite support
- **macOS:** Binaries are code-signed automatically
- **Windows:** Built with MinGW cross-compilation
- **FreeBSD:** Static binary (CGO disabled)

## Build Configuration

### GoReleaser Configuration

**File:** `.goreleaser.yml`

**Build Targets:**
- 6 platform/architecture combinations
- Customized ldflags for version injection
- Cross-compilation for Windows (MinGW)
- Cross-compilation for Linux ARM (aarch64-linux-gnu-gcc)

**Key Settings:**
```yaml
builds:
  - env:
      - CGO_ENABLED=1
    ldflags:
      - -s -w  # Strip debug info
      - -X github.com/steveyegge/gastown/internal/cmd.Version={{.Version}}
      - -X github.com/steveyegge/gastown/internal/cmd.Build={{.ShortCommit}}
      - -X github.com/steveyegge/gastown/internal/cmd.Commit={{.Commit}}
      - -X github.com/steveyegge/gastown/internal/cmd.Branch={{.Branch}}
```

### Archive Format
- **Unix:** tar.gz
- **Windows:** zip
- **Includes:** Binary, LICENSE, README.md
- **Naming:** `gastown_<version>_<os>_<arch>.tar.gz`

### Checksums
- **Algorithm:** SHA256
- **File:** `checksums.txt`
- **Purpose:** Verify download integrity

## CI/CD Pipeline

### GitHub Actions Workflows

#### 1. CI Workflow (`.github/workflows/ci.yml`)
**Triggers:** Push, Pull Request
**Purpose:** Main continuous integration
**Steps:**
- Checkout code
- Setup Go environment
- Install dependencies
- Run linters (golangci-lint v2.9.0)
- Run unit tests
- Build binary

#### 2. E2E Tests (`.github/workflows/e2e.yml`)
**Triggers:** Push, Pull Request
**Purpose:** End-to-end testing
**Steps:**
- Build Docker image (Dockerfile.e2e)
- Run E2E test suite in container
- Report results

#### 3. Integration Tests (`.github/workflows/integration.yml`)
**Triggers:** Push, Pull Request
**Purpose:** Integration testing
**Steps:**
- Setup test environment
- Run integration test suite
- Verify cross-package functionality

#### 4. Windows CI (`.github/workflows/windows-ci.yml`)
**Triggers:** Push, Pull Request
**Purpose:** Windows-specific validation
**Steps:**
- Setup Windows environment
- Build Windows binary
- Run Windows-specific tests

#### 5. Release Workflow (`.github/workflows/release.yml`)
**Triggers:** Tag push (v*.*.*)
**Purpose:** Automated release
**Steps:**
- Run GoReleaser
- Build all platform binaries
- Generate changelog
- Create GitHub release
- Upload artifacts
- Publish to GitHub Releases

#### 6-10. Issue Management Workflows
- **triage-label.yml** - Auto-label new issues
- **close-stale-needs.yml** - Close stale issues
- **remove-needs-info.yml** - Label automation
- **remove-needs-triage.yml** - Label automation
- **block-internal-prs.yml** - PR validation

## Release Process

### Automated Release

1. **Tag Creation**
   ```bash
   git tag -a v1.2.3 -m "Release v1.2.3"
   git push origin v1.2.3
   ```

2. **Automatic Steps** (via GitHub Actions + GoReleaser)
   - Run full CI pipeline
   - Build binaries for all platforms
   - Generate changelog from conventional commits
   - Create GitHub release
   - Upload binaries and checksums
   - Publish release notes

3. **Manual NPM Publish** (separate process)
   ```bash
   cd npm-package
   npm version 1.2.3
   npm publish
   ```

### Changelog Generation

**Auto-Generated from Commits:**
- Groups by type (Features, Bug Fixes, Others)
- Excludes docs, test, and chore commits
- Follows conventional commit format

**Format:**
```markdown
## Features
- feat: add new feature

## Bug Fixes
- fix: resolve issue

## Others
- Other changes
```

## Distribution Channels

### 1. GitHub Releases

**URL:** `https://github.com/steveyegge/gastown/releases`

**Assets per Release:**
- 6 platform binaries (tar.gz/zip)
- checksums.txt
- Source code (auto-generated by GitHub)

**Installation:**
```bash
# Download for your platform
wget https://github.com/steveyegge/gastown/releases/download/v0.7.0/gastown_0.7.0_darwin_arm64.tar.gz

# Extract
tar -xzf gastown_0.7.0_darwin_arm64.tar.gz

# Move to PATH
mv gt /usr/local/bin/
```

### 2. Homebrew (macOS/Linux)

**Formula Location:** Managed separately (homebrew-gastown tap)

**Installation:**
```bash
brew install gastown
```

**Update:**
```bash
brew upgrade gastown
```

### 3. NPM (Node.js Ecosystem)

**Package:** `@gastown/gt`
**Registry:** npmjs.com

**Installation:**
```bash
npm install -g @gastown/gt
```

**Update:**
```bash
npm update -g @gastown/gt
```

**NPM Package Structure:**
- Postinstall script downloads/installs Go binary
- Platform detection (darwin/linux/win32)
- Architecture detection (x64/arm64)
- Wrapper script (`bin/gt.js`)

## Docker (E2E Testing Only)

**Note:** Docker is used for E2E testing only, not for deployment.

**Dockerfile:** `Dockerfile.e2e`
**Purpose:** Isolated E2E test environment

**Build:**
```bash
docker build -f Dockerfile.e2e -t gastown-test .
```

**Run Tests:**
```bash
docker run --rm gastown-test
```

## Environment Requirements

### Production (End Users)

**Minimal Requirements:**
- OS: Linux, macOS, Windows, or FreeBSD
- Git 2.25+
- sqlite3 (usually pre-installed)
- beads (bd) 0.44.0+ for formula support

**Optional:**
- tmux 3.0+ for enhanced session management
- AI runtime (Claude Code, Codex, etc.)

### Development

**Additional Requirements:**
- Go 1.24.2+
- Make
- golangci-lint (for linting)
- Docker (for E2E tests)

### CI/CD

**GitHub Actions Runner Requirements:**
- Ubuntu, macOS, and Windows runners
- Go environment
- Cross-compilation toolchains (MinGW, aarch64-linux-gnu)
- Docker (for E2E)

## Infrastructure

### Repository Hosting
**Platform:** GitHub
**URL:** `https://github.com/steveyegge/gastown`
**Features:**
- Version control (Git)
- Issue tracking
- Pull requests
- GitHub Actions (CI/CD)
- GitHub Releases (binary distribution)

### Binary Distribution
**GitHub Releases:** Primary distribution
**Homebrew:** Secondary (macOS/Linux)
**NPM:** Secondary (Node.js ecosystem)

### No External Services Required
- No cloud hosting
- No external databases
- No SaaS dependencies
- Fully self-contained deployment

## Installation Verification

After installation, verify:

```bash
# Check installation
gt --version

# Should output version info
# Example: gt version 0.7.0 (commit abc1234, built 2026-02-17T00:00:00Z)

# Check help
gt --help

# Create test workspace
gt install ~/gt-test --git
```

## Upgrade Process

### Homebrew
```bash
brew upgrade gastown
```

### NPM
```bash
npm update -g @gastown/gt
```

### Manual (GitHub Releases)
1. Download new version binary
2. Replace existing binary in PATH
3. Verify: `gt --version`

## Rollback Process

### Homebrew
```bash
# View versions
brew info gastown

# Install specific version
brew install gastown@0.6.0
```

### NPM
```bash
npm install -g @gastown/gt@0.6.0
```

### Manual
Download and install previous version from GitHub Releases

## Security Considerations

### Code Signing
- **macOS:** Binaries automatically signed during build
- **Windows:** Not currently signed (consider for future)

### Checksum Verification
```bash
# Download binary and checksums.txt
wget https://github.com/steveyegge/gastown/releases/download/v0.7.0/gastown_0.7.0_darwin_arm64.tar.gz
wget https://github.com/steveyegge/gastown/releases/download/v0.7.0/checksums.txt

# Verify checksum
sha256sum -c checksums.txt --ignore-missing
```

### Supply Chain Security
- **Go Modules:** Checksums in go.sum
- **GitHub Actions:** Pinned action versions recommended
- **Dependencies:** Regular updates via Dependabot

## Monitoring & Observability

**Note:** Gas Town is a CLI tool with no telemetry or analytics.

**Logging:**
- Local logs only (no remote logging)
- User-controlled log levels
- No data collection

**Updates:**
- Users must manually update
- No auto-update mechanism (by design)

## Troubleshooting

### Issue: Binary not found after install
**Solution:** Verify PATH includes installation directory

### Issue: Permission denied
**Solution:**
```bash
chmod +x /path/to/gt
```

### Issue: Version mismatch
**Solution:** Clear old installations:
```bash
which -a gt  # Find all installations
# Remove stale versions
```

### Issue: NPM install fails
**Solution:** Check Node.js version (requires 14+)

## Future Deployment Enhancements

**Potential Improvements:**
- Windows code signing
- Auto-update mechanism (optional)
- Metrics/telemetry (opt-in)
- Container distribution (Docker Hub)
- Additional package managers (Chocolatey, Scoop, etc.)
