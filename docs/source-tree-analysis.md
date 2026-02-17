# Gas Town - Source Tree Analysis

**Generated:** 2026-02-17
**Repository Type:** Monolith with NPM wrapper

## Annotated Directory Structure

```
gastown/
├── cmd/                          # Application entry points
│   └── gt/
│       ├── main.go              # ⚡ ENTRY POINT - Delegates to internal/cmd.Execute()
│       └── build_test.go        # Build verification test
│
├── internal/                     # Core implementation (58 packages, 682 Go files)
│   ├── cmd/                     # 🎯 CLI command implementations (50+ commands)
│   │   ├── *.go                 # Cobra command handlers
│   │   └── config/              # Command-specific configuration
│   │
│   ├── agent/                   # 🤖 Agent lifecycle & provider abstraction
│   │   ├── provider.go          # Provider interface (Claude, Codex, etc.)
│   │   ├── session.go           # Session management
│   │   └── spawn.go             # Agent spawning logic
│   │
│   ├── hooks/                   # 🪝 Git-backed persistence (worktrees)
│   │   ├── registry.go          # Hook registry
│   │   ├── worktree.go          # Git worktree operations
│   │   └── state.go             # State management
│   │
│   ├── convoy/                  # 🚚 Work tracking system
│   │   ├── manager.go           # Convoy management
│   │   ├── database.go          # SQLite operations
│   │   └── tracker.go           # Progress tracking
│   │
│   ├── mail/                    # 📧 Inter-agent mail protocol
│   │   ├── mailbox.go           # Mailbox operations
│   │   ├── protocol.go          # Message format
│   │   └── identity.go          # Mail identities
│   │
│   ├── formula/                 # ⚙️ Workflow automation engine
│   │   ├── parser.go            # TOML formula parsing
│   │   ├── engine.go            # Step execution
│   │   ├── variables.go         # Variable substitution
│   │   └── formulas/            # Built-in formulas (33 TOML files)
│   │
│   ├── mayor/                   # 🎩 AI coordinator daemon
│   │   ├── coordinator.go       # Task coordination
│   │   └── context.go           # Context management
│   │
│   ├── polecat/                 # 🦨 Worker agent implementation
│   │   ├── lifecycle.go         # Agent lifecycle
│   │   └── execution.go         # Work execution
│   │
│   ├── deacon/                  # ⏰ Background task daemon
│   │   └── scheduler.go         # Task scheduling
│   │
│   ├── witness/                 # 👁️ Monitoring daemon
│   │   └── monitor.go           # Health monitoring
│   │
│   ├── refinery/                # 🔄 Data processing daemon
│   │   └── processor.go         # Data cleanup
│   │
│   ├── beads/                   # 📿 Beads integration
│   │   ├── client.go            # Beads API client
│   │   └── types.go             # Custom type support
│   │
│   ├── runtime/                 # 🔌 Runtime abstraction
│   │   ├── provider.go          # Provider interface
│   │   └── registry.go          # Runtime registry
│   │
│   ├── config/                  # ⚙️ Configuration management
│   │   ├── loader.go            # Config loading
│   │   ├── layers.go            # Layered config
│   │   └── roles/               # Role definitions (7 TOML files)
│   │       ├── mayor.toml       # Mayor role config
│   │       ├── polecat.toml     # Polecat role config
│   │       ├── deacon.toml      # Deacon role config
│   │       ├── witness.toml     # Witness role config
│   │       ├── refinery.toml    # Refinery role config
│   │       ├── crew.toml        # Crew role config
│   │       └── dog.toml         # Dog role config
│   │
│   ├── rig/                     # 🏗️ Project container management
│   │   ├── manager.go           # Rig operations
│   │   └── config.go            # Per-rig configuration
│   │
│   ├── crew/                    # 👤 Crew workspace
│   │   └── workspace.go         # Workspace setup
│   │
│   ├── workspace/               # 🏘️ Town workspace
│   │   ├── town.go              # Town operations
│   │   └── init.go              # Initialization
│   │
│   ├── git/                     # 📚 Git operations
│   │   ├── worktree.go          # Worktree management
│   │   ├── branch.go            # Branch operations
│   │   └── commit.go            # Commit operations
│   │
│   ├── tui/                     # 🖥️ Terminal UI (Bubbletea)
│   │   ├── models/              # Bubbletea models
│   │   ├── components/          # Reusable components
│   │   └── views/               # View rendering
│   │
│   ├── web/                     # 🌐 Web dashboard
│   │   ├── server.go            # HTTP server
│   │   ├── handlers/            # Route handlers
│   │   └── templates/           # HTML templates
│   │
│   ├── events/                  # 📡 Event system
│   │   ├── emitter.go           # Event emission
│   │   └── handlers.go          # Event handlers
│   │
│   ├── state/                   # 💾 State management
│   │   ├── global.go            # Global state
│   │   └── persist.go           # State persistence
│   │
│   ├── session/                 # 🔗 Session management
│   │   ├── manager.go           # Session lifecycle
│   │   └── recovery.go          # Crash recovery
│   │
│   ├── boot/                    # 🚀 System bootstrap
│   │   └── init.go              # Initialization sequences
│   │
│   ├── daemon/                  # 🔄 Daemon utilities
│   │   ├── lifecycle.go         # Daemon management
│   │   └── supervisor.go        # Process supervision
│   │
│   ├── protocol/                # 📋 Protocol definitions
│   │   └── messages.go          # Message formats
│   │
│   ├── claude/                  # Claude Code integration
│   ├── gemini/                  # Gemini integration
│   ├── opencode/                # OpenAI Codex integration
│   ├── copilot/                 # Copilot integration
│   │
│   ├── tmux/                    # 📺 Tmux integration
│   │   ├── session.go           # Session management
│   │   └── pane.go              # Pane operations
│   │
│   ├── lock/                    # 🔒 File locking
│   │   └── flock.go             # Concurrent access control
│   │
│   ├── mq/                      # 📬 Message queue
│   │   ├── queue.go             # Queue operations
│   │   └── persist.go           # Message persistence
│   │
│   ├── templates/               # 📄 Template system
│   │   └── agents/              # Agent templates
│   │
│   ├── style/                   # 🎨 Terminal styling
│   │   ├── colors.go            # Color schemes
│   │   └── themes.go            # Theme definitions
│   │
│   ├── ui/                      # 🖼️ UI utilities
│   │   └── formatters.go        # Output formatting
│   │
│   ├── util/                    # 🛠️ General utilities
│   │   └── helpers.go           # Helper functions
│   │
│   ├── townlog/                 # 📝 Logging system
│   │   └── logger.go            # Structured logging
│   │
│   ├── version/                 # 🏷️ Version management
│   │   └── version.go           # Version info
│   │
│   ├── doctor/                  # 🏥 Diagnostic tool
│   │   └── checks.go            # Health checks
│   │
│   ├── plugin/                  # 🔌 Plugin system
│   │   ├── loader.go            # Plugin loading
│   │   └── api.go               # Plugin API
│   │
│   ├── deps/                    # 📦 Dependency management
│   ├── suggest/                 # 💡 Suggestion engine
│   ├── swarm/                   # 🐝 Multi-agent swarm
│   ├── keepalive/               # 💓 Keepalive mechanisms
│   ├── nudge/                   # 🔔 Notification system
│   ├── feed/                    # 📰 Activity feed
│   ├── wisp/                    # 🌬️ Lightweight protocol
│   ├── wasteland/               # 🏜️ Orphaned work management
│   ├── dog/                     # 🐕 Watchdog functionality
│   ├── activity/                # 📊 Activity tracking
│   ├── connection/              # 🔗 Connection management
│   ├── checkpoint/              # ✅ State checkpoints
│   ├── shell/                   # 🐚 Shell operations
│   ├── constants/               # 📌 Application constants
│   ├── cli/                     # 🖱️ CLI utilities
│   ├── wrappers/                # 🎁 External tool wrappers
│   ├── krc/                     # (Unknown component)
│   └── doltserver/              # Dolt database (deprecated)
│
├── npm-package/                  # 📦 NPM distribution wrapper
│   ├── package.json             # NPM package manifest
│   ├── bin/
│   │   └── gt.js                # ⚡ ENTRY POINT - Wrapper script
│   ├── scripts/
│   │   ├── postinstall.js       # Binary installation
│   │   └── test.js              # Test script
│   └── README.md                # NPM package docs
│
├── docs/                         # 📚 Documentation (42+ markdown files)
│   ├── README.md                # Project overview (auto-generated)
│   ├── concepts/                # Core concepts (6 files)
│   │   ├── convoy.md            # Convoy system
│   │   ├── identity.md          # Agent identity
│   │   ├── molecules.md         # Formula instances
│   │   ├── polecat-lifecycle.md # Worker lifecycle
│   │   ├── propulsion-principle.md # Git-backed storage
│   │   └── integration-branches.md # Integration workflow
│   ├── design/                  # Architecture design (17 files)
│   │   ├── architecture.md      # System architecture
│   │   ├── agent-provider-interface.md
│   │   ├── mail-protocol.md     # Inter-agent mail
│   │   ├── plugin-system.md     # Plugin architecture
│   │   ├── federation.md        # Multi-town federation
│   │   └── ...                  # Additional design docs
│   ├── examples/
│   │   └── hanoi-demo.md        # Tower of Hanoi demo
│   └── issues/                  # Issue documentation
│
├── .beads/                       # Beads integration
│   ├── formulas/                # Formula definitions (33 TOML files)
│   │   ├── gastown-release.formula.toml
│   │   ├── mol-polecat-work.formula.toml
│   │   ├── towers-of-hanoi.formula.toml
│   │   └── ...                  # More formulas
│   └── README.md
│
├── scripts/                      # Build and utility scripts
│   └── migration-test/          # Migration testing
│
├── templates/                    # Configuration templates
│   └── agents/                  # Agent role templates
│
├── .github/                      # GitHub configuration
│   └── workflows/               # CI/CD workflows (10 YAML files)
│       ├── ci.yml               # Main CI pipeline
│       ├── e2e.yml              # E2E tests
│       ├── integration.yml      # Integration tests
│       ├── release.yml          # Release automation
│       └── ...                  # Additional workflows
│
├── go.mod                        # 🔧 Go module definition
├── go.sum                        # 🔒 Dependency checksums
├── Makefile                      # 🔨 Build automation
├── .goreleaser.yml              # 📦 Multi-platform release config
├── .golangci.yml                # 🔍 Linter configuration
├── Dockerfile.e2e               # 🐳 E2E test container
│
├── README.md                     # 📖 Main project README
├── CONTRIBUTING.md              # 🤝 Contribution guidelines
├── AGENTS.md                    # 🤖 Agent documentation
├── CHANGELOG.md                 # 📝 Version history
├── CODE_OF_CONDUCT.md           # 📜 Code of conduct
├── RELEASING.md                 # 🚀 Release process
├── SECURITY.md                  # 🔐 Security policies
└── LICENSE                       # ⚖️ MIT License
```

## Entry Points

### Primary Entry Point
**File:** `cmd/gt/main.go`
**Purpose:** Application bootstrap - delegates to Cobra command tree
**Execution Flow:** `main()` → `internal/cmd.Execute()` → Cobra command routing

### NPM Wrapper Entry Point
**File:** `npm-package/bin/gt.js`
**Purpose:** Node.js wrapper that invokes the Go binary
**Execution Flow:** `gt.js` → Platform detection → Execute Go binary

## Critical Folders Summary

| Folder | Purpose | Key Files | Integration Points |
|--------|---------|-----------|-------------------|
| **cmd/gt/** | Application entry | main.go | → internal/cmd |
| **internal/cmd/** | CLI commands | 50+ command files | → All internal packages |
| **internal/agent/** | Agent management | provider.go, session.go | → runtime, hooks, mail |
| **internal/hooks/** | Persistence layer | registry.go, worktree.go | → git, state |
| **internal/convoy/** | Work tracking | manager.go, database.go | → beads, SQLite |
| **internal/mail/** | Communication | mailbox.go, protocol.go | → agent, events |
| **internal/formula/** | Workflow automation | engine.go, parser.go | → beads, convoy |
| **internal/config/** | Configuration | loader.go, roles/ | → All packages |
| **internal/runtime/** | Runtime abstraction | provider.go | → claude, gemini, etc. |
| **internal/tui/** | Terminal UI | models/, views/ | → style, ui |
| **internal/web/** | Web dashboard | server.go, handlers/ | → convoy, agent, hooks |
| **npm-package/** | Distribution | package.json, scripts/ | → Go binary |
| **.beads/** | Workflow formulas | formulas/ (33 TOML) | → internal/formula |
| **docs/** | Documentation | 42 markdown files | N/A |

## Package Dependency Overview

```
User Interface Layer
    ├─ internal/cmd (CLI commands)
    ├─ internal/tui (Terminal UI)
    └─ internal/web (Web dashboard)
            ↓
Orchestration Layer
    ├─ internal/mayor (AI coordinator)
    ├─ internal/convoy (Work tracking)
    ├─ internal/formula (Workflows)
    └─ internal/mail (Communication)
            ↓
Core Domain Layer
    ├─ internal/agent (Agent management)
    ├─ internal/polecat (Workers)
    ├─ internal/hooks (Persistence)
    ├─ internal/beads (Issue tracking)
    └─ internal/runtime (Runtime abstraction)
            ↓
Infrastructure Layer
    ├─ internal/git (Git operations)
    ├─ internal/config (Configuration)
    ├─ internal/state (State management)
    ├─ internal/session (Sessions)
    └─ internal/events (Event system)
            ↓
Utilities Layer
    ├─ internal/util (Utilities)
    ├─ internal/style (Styling)
    ├─ internal/townlog (Logging)
    └─ internal/lock (Locking)
```

## Multi-Part Structure

### Part 1: gastown-cli (Primary)
**Root:** `/Users/jonathangreen/Documents/gastown`
**Key Directories:** cmd/, internal/, docs/, .beads/, scripts/
**Technologies:** Go 1.24.2, Cobra, Bubbletea, SQLite

### Part 2: npm-wrapper (Distribution)
**Root:** `/Users/jonathangreen/Documents/gastown/npm-package`
**Key Directories:** bin/, scripts/
**Technologies:** Node.js 14+, NPM
**Integration:** → Wraps and distributes the Go binary

## Key Files Reference

### Configuration
- `internal/config/roles/*.toml` - Role definitions (7 files)
- `.golangci.yml` - Linter config
- `.goreleaser.yml` - Release config
- `Makefile` - Build config

### Entry Points
- `cmd/gt/main.go` - Go binary entry
- `npm-package/bin/gt.js` - NPM wrapper entry
- `internal/cmd/*.go` - Command implementations

### Workflow Definitions
- `.beads/formulas/*.formula.toml` - 33 formula definitions
- `internal/formula/formulas/*.formula.toml` - Same (duplicated)

### Documentation
- `docs/design/*.md` - 17 design documents
- `docs/concepts/*.md` - 6 concept documents
- Root `*.md` files - Project documentation

### CI/CD
- `.github/workflows/*.yml` - 10 GitHub Actions workflows
- `Dockerfile.e2e` - E2E test container

## Notes

- **Modular Architecture:** 58 packages with clear separation of concerns
- **Git as Database:** Innovative use of git worktrees for persistence
- **Multi-Runtime Support:** Abstracted providers for different AI backends
- **Comprehensive Documentation:** 42+ markdown files covering all aspects
- **Test Coverage:** 268 test files (39% ratio) across codebase
- **Formula System:** 33 workflow definitions for common tasks
- **Daemon Processes:** 4 background daemons (mayor, deacon, witness, refinery)
