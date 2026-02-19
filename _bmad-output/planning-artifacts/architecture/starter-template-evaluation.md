# Starter Template Evaluation

## Primary Technology Domain

**CLI tool with daemon processes** - Go-based brownfield integration into existing gastown architecture.

## "Starter" Foundation: Existing Gastown Architecture

**Rationale for Working Within Existing Architecture:**

This is a brownfield integration project, not a greenfield project. The "starter template" is the existing gastown codebase (v0.7.0) with its established patterns and conventions. We must integrate crosstown functionality while:

- Preserving all existing commands and workflows
- Following established architectural patterns
- Maintaining backward compatibility
- Respecting existing git-backed persistence model
- Adhering to existing build and test requirements

**Existing Architectural Foundation:**

**Language & Runtime:**
- Go 1.24.2 (strict requirement, cannot upgrade)
- CGO enabled for most platforms (disabled for FreeBSD)
- Module-based dependency management

**CLI Framework:**
- Cobra v1.10.2 for command structure
- 50+ existing commands organized by domain
- Flag-based configuration at each level
- Shell completion support (bash, zsh, fish, PowerShell)

**TUI Framework:**
- Bubbletea v1.3.10 with Elm/MVU architecture
- Lipgloss v1.1.1 for terminal styling
- Immutable models, pure update functions
- Message-driven state updates

**Persistence Layer:**
- Git worktrees as primary persistence ("Propulsion Principle")
- SQLite embedded database for convoy queries
- TOML configuration files
- Git commits provide audit trail and crash recovery

**Build Tooling:**
- Makefile-based build orchestration
- Version info injected via ldflags
- GoReleaser v2 for multi-platform releases
- Automatic macOS codesigning

**Testing Infrastructure:**
- Unit tests: `*_test.go` alongside source
- E2E tests: ONLY in Docker containers (`make test-e2e-container`)
- Integration tests: Separate with build tags
- golangci-lint v2.9.0 (pinned)

**Code Organization:**
- Entry point: `cmd/gt/main.go` only
- All implementation: `internal/` (58 packages by domain)
- Commands: `internal/cmd/` with Cobra structure
- Domain packages: `internal/<domain>/`

**Development Patterns Established:**
- Command Pattern (Cobra CLI tree)
- MVU Architecture (Bubbletea)
- Plugin Architecture (agent providers)
- Event-Driven Coordination (mail protocol)
- Git-Backed Persistence (hooks)

**Integration Points for Crosstown:**

New packages to add:
- `internal/ilp/` - ILP client, BTP protocol, payment channels
- `internal/nostr/` - NIP-01 WebSocket, TOON encoding/decoding
- `internal/crosstown/` - Integration layer, channel manager, job correlation

New commands to add:
- `internal/cmd/crosstown.go` - New command group for crosstown operations

Dashboard extension:
- Extend `internal/mayor/dashboard.go` with crosstown panel following Bubbletea MVU patterns

Configuration extension:
- Add crosstown settings to existing TOML config system
- Support environment variables following existing `GASTOWN_*` prefix pattern

**Note:** Rather than project initialization, the first implementation story will be setting up the new package structure (`internal/ilp/`, `internal/nostr/`, `internal/crosstown/`) with stub implementations following existing gastown patterns.

