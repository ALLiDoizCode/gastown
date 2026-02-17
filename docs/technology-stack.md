# Gas Town - Technology Stack

**Generated:** 2026-02-17
**Workflow Version:** 1.2.0

## Part 1: gastown-cli (Primary)

### Core Technologies

| Category | Technology | Version | Justification |
|----------|-----------|---------|---------------|
| **Language** | Go | 1.24.2 | High-performance compiled language with excellent concurrency support |
| **CLI Framework** | Cobra | v1.10.2 | Industry-standard CLI framework for Go with subcommands, flags, and completions |
| **TUI Framework** | Bubbletea | v1.3.10 | Elm-inspired terminal UI framework for interactive CLI applications |
| **Terminal Styling** | Lipgloss | v1.1.1 | Styling library for terminal UIs with themes and layouts |
| **Markdown Rendering** | Glamour | v0.10.0 | Markdown rendering in terminal |
| **Browser Automation** | Rod | v0.116.2 | Chrome DevTools Protocol automation for web dashboard |
| **File Locking** | gofrs/flock | v0.13.0 | Cross-platform file locking for concurrent access control |
| **UUID Generation** | google/uuid | v1.6.0 | Unique identifier generation for agents, convoys, and sessions |
| **TOML Parser** | BurntSushi/toml | v1.6.0 | Configuration and formula file parsing |
| **Terminal Support** | golang.org/x/term | v0.38.0 | Low-level terminal control and ANSI support |

### Architecture & Patterns

| Category | Technology | Description |
|----------|-----------|-------------|
| **Database** | SQLite | Convoy database for work tracking queries |
| **Version Control** | Git Worktrees | Persistent storage mechanism for agent state (hooks) |
| **Configuration** | TOML | Formula definitions, role configurations |
| **Build System** | Make + GoReleaser | Multi-platform build and release automation |
| **Testing** | Go test + Docker | Unit tests and isolated E2E container tests |

### Build & Deployment

| Category | Technology | Version/Details |
|----------|-----------|-----------------|
| **Build Tool** | Make | Makefile-based build orchestration |
| **Release Tool** | GoReleaser | v2 - Multi-platform binary distribution |
| **CI/CD** | GitHub Actions | 10 workflows (ci, e2e, integration, release, triage, etc.) |
| **Linting** | golangci-lint | v2.9.0 (pinned for consistency) |
| **Code Signing** | macOS codesign | Automatic signing on macOS builds |

### Platform Support

| Platform | Architectures | CGO | Notes |
|----------|--------------|-----|-------|
| **Linux** | amd64, arm64 | Enabled | Primary platform |
| **macOS** | amd64, arm64 | Enabled | Apple Silicon support |
| **Windows** | amd64 | Enabled | MinGW cross-compilation |
| **FreeBSD** | amd64 | Disabled | Static binary |

### Key Dependencies (Selected)

```
github.com/spf13/cobra v1.10.2           # CLI framework
github.com/charmbracelet/bubbletea v1.3.10  # TUI framework
github.com/charmbracelet/lipgloss v1.1.1    # Terminal styling
github.com/charmbracelet/glamour v0.10.0    # Markdown rendering
github.com/go-rod/rod v0.116.2              # Browser automation
github.com/gofrs/flock v0.13.0              # File locking
github.com/google/uuid v1.6.0               # UUID generation
github.com/BurntSushi/toml v1.6.0          # TOML parsing
```

## Part 2: npm-wrapper (Distribution Package)

### Core Technologies

| Category | Technology | Version | Justification |
|----------|-----------|---------|---------------|
| **Runtime** | Node.js | 14+ | Widely available runtime for cross-platform distribution |
| **Package Manager** | NPM | - | Standard JavaScript package distribution |
| **Package Name** | @gastown/gt | 0.7.0 | Scoped npm package for the Gas Town CLI |

### Build & Distribution

| Category | Technology | Description |
|----------|-----------|-------------|
| **Postinstall** | Node.js script | Binary installation and platform detection |
| **Platform Support** | darwin, linux, win32 | Cross-platform wrapper |
| **Architecture Support** | x64, arm64 | Multi-architecture support |
| **Binary Wrapper** | bin/gt.js | Executable wrapper that invokes Go binary |

### Integration

The NPM package:
- Downloads or references the appropriate Go binary for the user's platform
- Provides `gt` command via npm bin scripts
- Handles postinstall setup and platform detection
- Acts as a thin wrapper for the compiled Go application

## Architecture Patterns

### Primary Architecture: CLI + TUI with Git-Backed Persistence

**Pattern Classification:** Command-line tool with terminal UI and daemon processes

**Key Architectural Patterns:**

1. **Command Pattern**
   - Cobra command tree for CLI operations
   - Subcommands: `rig`, `crew`, `mayor`, `convoy`, `sling`, `config`, etc.
   - Flag-based configuration

2. **Model-View-Update (Elm Architecture)**
   - Bubbletea TUI components use MVU pattern
   - Message-driven state updates
   - Immutable state transformations

3. **Git-Backed Persistence ("Propulsion Principle")**
   - Git worktrees as persistent storage (hooks)
   - Version-controlled agent state
   - Survives process crashes and restarts
   - Multi-agent coordination through git

4. **Plugin Architecture**
   - Agent provider abstraction layer
   - Support for multiple runtimes (Claude, Codex, Cursor, Gemini, etc.)
   - Configurable agent commands and args

5. **Event-Driven Coordination**
   - Mail protocol for inter-agent communication
   - Convoy-based work tracking
   - Asynchronous agent spawning and monitoring

6. **Daemon Processes**
   - Mayor: AI coordinator
   - Deacon: Background task manager
   - Witness: Monitoring and oversight
   - Refinery: Data processing and cleanup

7. **Formula System (Workflow Automation)**
   - TOML-based workflow definitions
   - Variable substitution and step dependencies
   - Integration with Beads issue tracking

### Secondary Architecture: Distribution Wrapper

**Pattern:** NPM binary wrapper for cross-platform Go application distribution

## External Integrations

| System | Purpose | Integration Type |
|--------|---------|-----------------|
| **Beads** | Issue tracking and workflow automation | Native integration (git-backed) |
| **Claude Code** | Primary AI agent runtime | Process spawning + hooks |
| **Codex CLI** | Alternative AI runtime | Process spawning |
| **Cursor** | Alternative AI runtime | Process spawning |
| **Gemini** | Alternative AI runtime | Process spawning |
| **Git** | Version control + persistent storage | Git worktrees, hooks |
| **tmux** | Session management (optional) | Process integration |
| **SQLite** | Convoy query database | Embedded database |

## Development Workflow

### Build Commands

```bash
make build          # Build binary with version info
make install        # Install to ~/.local/bin
make test           # Run Go tests
make test-e2e-container  # Run E2E tests in Docker
make clean          # Clean build artifacts
```

### Release Process

- GoReleaser handles multi-platform builds
- GitHub Actions automates releases
- Changelog auto-generated from conventional commits
- Binaries published to GitHub Releases
- NPM package published separately

## CI/CD Workflows

| Workflow | Purpose | Trigger |
|----------|---------|---------|
| **ci.yml** | Main CI pipeline | Push, PR |
| **e2e.yml** | End-to-end tests | Push, PR |
| **integration.yml** | Integration tests | Push, PR |
| **windows-ci.yml** | Windows-specific CI | Push, PR |
| **release.yml** | Release automation | Tag push |
| **triage-label.yml** | Auto-label issues | Issue creation |
| **close-stale-needs.yml** | Stale issue cleanup | Schedule |
| **remove-needs-info.yml** | Label automation | Issue comment |
| **remove-needs-triage.yml** | Label automation | Issue comment |
| **block-internal-prs.yml** | PR validation | PR creation |

## Notable Design Choices

1. **Go for Performance**: Compiled language provides fast startup and execution for CLI tool
2. **Bubbletea for TUI**: Modern, reactive terminal UI framework with excellent developer experience
3. **Git Worktrees for Persistence**: Innovative use of git as persistent storage mechanism
4. **Multi-Runtime Support**: Abstracted agent providers allow flexibility in AI backend
5. **SQLite for Queries**: Embedded database for complex convoy queries without external dependencies
6. **TOML for Configuration**: Human-readable format for formulas and role definitions
7. **GoReleaser for Distribution**: Automated multi-platform builds with consistent release process
8. **NPM Wrapper**: Convenient distribution channel for Node.js users
