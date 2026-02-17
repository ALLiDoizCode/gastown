# Gas Town - Development Guide

**Generated:** 2026-02-17

## Prerequisites

### Required
- **Go 1.24.2+** - [go.dev/dl](https://go.dev/dl/)
- **Git 2.25+** - For worktree support
- **beads (bd) 0.44.0+** - [github.com/steveyegge/beads](https://github.com/steveyegge/beads)
  - Required for custom type support and formula system
- **sqlite3** - For convoy database queries
  - Usually pre-installed on macOS/Linux
  - Windows: Download from [sqlite.org](https://www.sqlite.org/download.html)

### Recommended
- **tmux 3.0+** - For full multi-session experience
- **Claude Code CLI** - Default AI runtime ([claude.ai/code](https://claude.ai/code))
- **make** - Build automation (usually pre-installed on Unix systems)

### Optional AI Runtimes
- **Codex CLI** - OpenAI alternative runtime
- **Cursor** - Cursor AI integration
- **Gemini** - Google Gemini integration

## Installation

### Option 1: Homebrew (macOS/Linux - Recommended)
```bash
brew install gastown
```

### Option 2: NPM (Cross-platform)
```bash
npm install -g @gastown/gt
```

### Option 3: From Source
```bash
# Clone repository
git clone https://github.com/steveyegge/gastown.git
cd gastown

# Install dependencies
go mod download

# Build binary
make build

# Install to ~/.local/bin (add to PATH if not already)
make install
```

### Option 4: Go Install (Not Recommended)
```bash
# Direct go install (not recommended - use make install instead)
go install github.com/steveyegge/gastown/cmd/gt@latest

# Note: make install is preferred as it includes version info and removes stale binaries
```

### Add to PATH
If installing from source, ensure `~/.local/bin` is in your PATH:
```bash
# Add to ~/.zshrc or ~/.bashrc
export PATH="$HOME/.local/bin:$PATH"
```

## Development Setup

### 1. Clone and Setup
```bash
git clone https://github.com/steveyegge/gastown.git
cd gastown
go mod download
```

### 2. Build
```bash
make build
```

**Build Output:** Binary at `./gt`

**Build Flags:** Version info injected via ldflags:
- `Version` - Git tag/commit
- `Commit` - Git commit hash
- `BuildTime` - Build timestamp
- `BuiltProperly` - Build verification flag

**macOS Code Signing:** Automatically signs binary on macOS

### 3. Run Tests
```bash
# Run all unit tests
make test

# Run with verbose output
go test -v ./...

# Run specific package
go test ./internal/agent

# Run with coverage
go test -cover ./...
```

### 4. Run E2E Tests
```bash
# E2E tests run in isolated Docker container (ONLY supported method)
make test-e2e-container
```

**Note:** E2E tests must run in Docker to ensure clean environment

### 5. Local Development
```bash
# Build and run
make build
./gt --help

# Or install locally
make install
gt --help
```

## Project Structure

```
gastown/
├── cmd/gt/              # Main entry point
├── internal/            # Core implementation (58 packages)
│   ├── cmd/            # CLI commands
│   ├── agent/          # Agent management
│   ├── hooks/          # Git-backed persistence
│   ├── convoy/         # Work tracking
│   └── ...             # 53 more packages
├── npm-package/         # NPM distribution
├── docs/               # Documentation
├── .beads/             # Beads formulas
└── scripts/            # Utility scripts
```

## Development Workflow

### Making Changes

1. **Create Branch**
   ```bash
   git checkout -b feature/my-feature
   ```

2. **Make Changes**
   - Edit code in `internal/` packages
   - Add tests for new functionality
   - Update documentation if needed

3. **Test Changes**
   ```bash
   make test
   make test-e2e-container  # If touching core functionality
   ```

4. **Build Locally**
   ```bash
   make build
   ./gt --version  # Verify build
   ```

5. **Lint Code**
   ```bash
   # golangci-lint is run automatically in CI
   # Install locally: https://golangci-lint.run/usage/install/
   golangci-lint run
   ```

6. **Commit Changes**
   ```bash
   git add .
   git commit -m "feat: add new feature"
   # Follow conventional commit format (see CONTRIBUTING.md)
   ```

7. **Push and Create PR**
   ```bash
   git push origin feature/my-feature
   # Create PR on GitHub
   ```

### Conventional Commits

Use conventional commit format for automatic changelog generation:
- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `test:` - Test additions/changes
- `chore:` - Maintenance tasks
- `refactor:` - Code refactoring

### Running Locally

```bash
# Create test workspace
gt install ~/gt-test --git
cd ~/gt-test

# Add a test project
gt rig add test-project /path/to/some/repo

# Create crew workspace
gt crew add dev --rig test-project
cd test-project/crew/dev

# Test commands
gt config show
gt agents
```

## Build Commands Reference

```bash
make build              # Build binary
make install            # Build and install to ~/.local/bin
make test               # Run unit tests
make test-e2e-container # Run E2E tests in Docker
make clean              # Remove build artifacts
make generate           # Run go generate
```

## Testing Strategy

### Unit Tests
- **Location:** `*_test.go` files alongside source
- **Run:** `make test` or `go test ./...`
- **Coverage:** 268 test files across codebase
- **Focus:** Individual package functionality

### Integration Tests
- **Location:** Separate test files with integration build tag
- **Run:** Via CI workflow
- **Focus:** Cross-package interactions

### E2E Tests
- **Location:** E2E test suite
- **Run:** `make test-e2e-container` (Docker required)
- **Focus:** Full workflow scenarios with real git operations

### Test Coverage
Current coverage: ~39% (268 test files / 682 source files)

## Code Generation

```bash
# Run all generators
make generate

# Or directly
go generate ./...
```

## Debugging

### Enable Verbose Logging
```bash
# Set log level
export GT_LOG_LEVEL=debug
gt <command>

# Or use flag (if available)
gt <command> --verbose
```

### Common Issues

**Issue:** `go: command not found`
**Solution:** Install Go from [go.dev/dl](https://go.dev/dl/)

**Issue:** Build fails with version info missing
**Solution:** Use `make build` instead of `go build` directly

**Issue:** E2E tests fail locally
**Solution:** Run via Docker: `make test-e2e-container`

**Issue:** Binary not found after install
**Solution:** Ensure `~/.local/bin` is in PATH, or check `~/go/bin`

## Configuration

### User-Level Config
`~/.config/gt/config.json`

### Town-Level Config
`~/gt/.gt/config.json` (in workspace)

### Rig-Level Config
`~/gt/<rig>/settings/config.json`

### Runtime Configuration Example
```json
{
  "runtime": {
    "provider": "claude",
    "command": "claude",
    "args": [],
    "prompt_mode": "none"
  }
}
```

## Environment Variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `GT_LOG_LEVEL` | Logging verbosity | `info` |
| `GT_TOWN_PATH` | Override town path | `~/gt` |
| `GT_CONFIG_PATH` | Override config location | `~/.config/gt` |

## Dependencies

### Go Module Dependencies
See `go.mod` for full list. Key dependencies:
- `github.com/spf13/cobra` - CLI framework
- `github.com/charmbracelet/bubbletea` - TUI
- `github.com/charmbracelet/lipgloss` - Styling
- `github.com/go-rod/rod` - Browser automation
- `github.com/gofrs/flock` - File locking
- `github.com/google/uuid` - UUID generation

### External Tool Dependencies
- **git** - Version control and persistence
- **beads (bd)** - Issue tracking and workflows
- **sqlite3** - Convoy queries
- **tmux** (optional) - Session management

## IDE Setup

### VS Code
Recommended extensions:
- Go (golang.go)
- Go Test Explorer
- Error Lens

### GoLand / IntelliJ IDEA
- Native Go support
- Import project from `go.mod`

## Common Development Tasks

### Adding a New Command
1. Create `internal/cmd/mycommand.go`
2. Implement command using Cobra
3. Register in root command
4. Add tests in `internal/cmd/mycommand_test.go`
5. Update documentation

### Adding a New Internal Package
1. Create `internal/mypackage/` directory
2. Implement package functionality
3. Add tests
4. Import in relevant commands
5. Document in package comment

### Adding a New Formula
1. Create `.beads/formulas/myformula.formula.toml`
2. Define steps and variables
3. Test with `bd cook myformula`
4. Document in formula comment

## Release Process

See `RELEASING.md` for detailed release instructions.

**Quick Overview:**
1. Update CHANGELOG.md
2. Create git tag
3. Push tag to trigger release workflow
4. GoReleaser builds multi-platform binaries
5. GitHub release created automatically
6. NPM package published separately

## Resources

- **Repository:** [github.com/steveyegge/gastown](https://github.com/steveyegge/gastown)
- **Documentation:** `docs/` directory
- **Issues:** [GitHub Issues](https://github.com/steveyegge/gastown/issues)
- **Contributing:** See `CONTRIBUTING.md`
- **Security:** See `SECURITY.md`
