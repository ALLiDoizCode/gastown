# Architecture: Gas Town CLI

**Part:** gastown-cli (Primary)
**Type:** CLI Tool with Daemons
**Language:** Go 1.24.2
**Generated:** 2026-02-17

## Executive Summary

Gas Town CLI is a sophisticated multi-agent orchestration system built in Go. It coordinates AI coding agents through an innovative git-backed persistence mechanism, providing reliable workflow automation with support for multiple AI runtimes. The architecture follows a modular, event-driven design with 58 internal packages organized into distinct layers.

## Technology Stack

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| **Language** | Go | 1.24.2 | Core implementation |
| **CLI Framework** | Cobra | v1.10.2 | Command routing |
| **TUI Framework** | Bubbletea | v1.3.10 | Terminal UI |
| **Styling** | Lipgloss | v1.1.1 | Terminal styling |
| **Browser Automation** | Rod | v0.116.2 | Web dashboard |
| **Database** | SQLite | Embedded | Convoy queries |
| **File Locking** | gofrs/flock | v0.13.0 | Concurrency control |
| **UUID** | google/uuid | v1.6.0 | Identifier generation |
| **Build** | Make + GoReleaser | v2 | Build automation |
| **CI/CD** | GitHub Actions | - | 10 workflows |

## Architecture Pattern

**Primary Pattern:** Layered CLI with Event-Driven Coordination

**Secondary Patterns:**
- Plugin Architecture (agent providers)
- Git-Backed Persistence ("Propulsion Principle")
- Daemon Processes (mayor, deacon, witness, refinery)
- Formula-Based Workflow Automation

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   USER INTERFACE LAYER                       │
│  ┌────────────┐  ┌────────────┐  ┌──────────────────────┐  │
│  │ CLI (Cobra)│  │ TUI (Tea)  │  │ Web Dashboard (Rod)  │  │
│  │ 50+ cmds   │  │ Interactive│  │ Real-time status     │  │
│  └────────────┘  └────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                 ORCHESTRATION LAYER                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Mayor   │  │  Convoy  │  │ Formula  │  │   Mail   │   │
│  │Coordinator│ │  Tracker │  │ Engine   │  │ Protocol │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                     CORE DOMAIN LAYER                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Agent   │  │ Polecat  │  │  Hooks   │  │  Beads   │   │
│  │ Provider │  │  Worker  │  │ Registry │  │Integration│  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                  INFRASTRUCTURE LAYER                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Git    │  │  Config  │  │  State   │  │  Events  │   │
│  │Worktrees │  │  Layers  │  │Management│  │  System  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                     PERSISTENCE LAYER                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │Git Worktrees │  │    SQLite    │  │  Agent Processes │  │
│  │   (Hooks)    │  │   (Convoy)   │  │(Claude/Codex/etc)│  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Component Overview

### User Interface Layer

#### CLI Commands (internal/cmd)
- **Purpose:** 50+ Cobra commands for all operations
- **Key Commands:** install, rig, crew, mayor, sling, convoy, config
- **Pattern:** Command pattern with subcommands

#### Terminal UI (internal/tui)
- **Purpose:** Interactive terminal interfaces
- **Framework:** Bubbletea (Elm architecture)
- **Components:** Models, Views, Update functions

#### Web Dashboard (internal/web)
- **Purpose:** Real-time web-based monitoring
- **Technology:** HTTP server with Rod browser automation
- **Features:** Agent status, convoy progress, hook visualization

### Orchestration Layer

#### Mayor (internal/mayor)
- **Purpose:** AI coordinator and task orchestrator
- **Responsibilities:**
  - Task breakdown and delegation
  - Context management
  - Convoy creation and management
- **Pattern:** Coordinator pattern

#### Convoy (internal/convoy)
- **Purpose:** Work tracking system
- **Database:** SQLite for complex queries
- **Features:**
  - Bundle multiple beads (tasks)
  - Track progress across agents
  - Report completion status

#### Formula Engine (internal/formula)
- **Purpose:** Workflow automation
- **Format:** TOML-defined workflows
- **Features:**
  - Variable substitution
  - Step dependencies (needs)
  - 33 built-in formulas

#### Mail Protocol (internal/mail)
- **Purpose:** Inter-agent communication
- **Components:**
  - Mailboxes (per-agent queues)
  - Identities
  - Groups
- **Pattern:** Message queue pattern

### Core Domain Layer

#### Agent Provider (internal/agent, internal/runtime)
- **Purpose:** Multi-runtime abstraction
- **Supported Runtimes:**
  - Claude Code (default)
  - Codex
  - Cursor
  - Gemini
  - Custom agents
- **Pattern:** Abstract factory + Strategy

#### Polecat (internal/polecat)
- **Purpose:** Worker agent implementation
- **Characteristics:**
  - Ephemeral sessions
  - Persistent identity
  - Work execution in hooks

#### Hooks Registry (internal/hooks)
- **Purpose:** Git-backed persistence management
- **Innovation:** Git worktrees as persistent storage
- **Benefits:**
  - Survives crashes
  - Version-controlled state
  - Rollback capability

#### Beads Integration (internal/beads)
- **Purpose:** Issue tracking integration
- **System:** Beads (git-backed issues)
- **Features:**
  - Bead ID handling (gt-abc12 format)
  - Custom types
  - Formula integration

### Infrastructure Layer

#### Git Operations (internal/git)
- **Purpose:** Git worktree and repository management
- **Operations:**
  - Worktree creation/deletion
  - Branch operations
  - Commit operations
  - Remote sync

#### Configuration (internal/config)
- **Purpose:** Layered configuration system
- **Layers:** Flags → Env → Rig → Town → User → Defaults
- **Files:** Role definitions (TOML), runtime configs

#### State Management (internal/state)
- **Purpose:** Application state coordination
- **Components:**
  - Global state
  - Per-agent state
  - State serialization

#### Events (internal/events)
- **Purpose:** Event-driven coordination
- **Pattern:** Publisher-subscriber
- **Use Cases:** Async notifications, inter-component communication

### Daemon Processes

#### Mayor Daemon
- **Purpose:** AI coordinator
- **Lifecycle:** User-initiated, persistent
- **Role:** Primary interface for users

#### Deacon Daemon
- **Purpose:** Background task execution
- **Lifecycle:** Auto-started with town
- **Role:** Periodic tasks, monitoring

#### Witness Daemon
- **Purpose:** Oversight and health monitoring
- **Lifecycle:** Auto-started with town
- **Role:** Health checks, alerting

#### Refinery Daemon
- **Purpose:** Data processing and cleanup
- **Lifecycle:** Periodic execution
- **Role:** Data normalization, garbage collection

## Data Architecture

### Persistent Data

#### Git Worktrees (Hooks)
```
.gt/
└── hooks/
    ├── agent-abc123/     # Git worktree
    │   ├── .git          # Points to main repo
    │   ├── state.json    # Agent state
    │   └── work/         # Work files
    └── agent-def456/
```

**Characteristics:**
- Version-controlled
- Rollback capable
- Survives crashes
- Distributed coordination

#### SQLite Database (Convoy)
```sql
-- Convoy tracking
CREATE TABLE convoys (
  id TEXT PRIMARY KEY,
  name TEXT,
  created_at TIMESTAMP,
  status TEXT
);

CREATE TABLE convoy_beads (
  convoy_id TEXT,
  bead_id TEXT,
  status TEXT
);
```

**Purpose:** Complex convoy queries

#### Configuration Files
- **TOML:** Role definitions, formulas
- **JSON:** Runtime config, rig settings
- **Environment:** `.env` files

### Data Flow Patterns

#### Work Assignment Flow
```
User/Mayor
    ↓ create convoy
Convoy Manager
    ↓ breakdown to beads
Beads (atomic tasks)
    ↓ sling
Agent assigned
    ↓ create hook
Hook (git worktree)
    ↓ execute
Work completed
    ↓ commit state
Hook persisted
    ↓ report
Convoy updated
```

#### Agent Lifecycle Flow
```
Spawn Request
    ↓
Provider Selection (claude/codex/etc)
    ↓
Hook Initialization (git worktree)
    ↓
Session Start
    ↓
Role Injection (CLAUDE.md)
    ↓
Mail Check (work assignment)
    ↓
Work Execution
    ↓
State Commit (to hook)
    ↓
Report Completion
    ↓
Session End (hook persists)
```

## API Design

**Note:** Gas Town is a CLI tool with no external API. Internal package APIs follow Go conventions.

### Key Internal Interfaces

#### Agent Provider Interface
```go
type Provider interface {
    Start(config Config) (Session, error)
    Stop(sessionID string) error
    SendMessage(sessionID, message string) error
}
```

#### Hook Registry Interface
```go
type Registry interface {
    Create(agentID string) (Hook, error)
    Get(agentID string) (Hook, error)
    List() ([]Hook, error)
    Delete(agentID string) error
}
```

## Component Inventory

### 58 Internal Packages

**By Domain:**

**Agent & Orchestration (8):**
agent, mayor, polecat, crew, daemon, deacon, witness, refinery

**Work Tracking (5):**
convoy, beads, formula, molecule, wasteland

**Communication (4):**
mail, events, mq, protocol

**Persistence & State (5):**
hooks, state, session, checkpoint, lock

**Infrastructure (8):**
git, config, runtime, boot, workspace, rig, tmux, shell

**User Interface (4):**
cmd, cli, tui, ui, web

**Utilities (10):**
util, style, townlog, version, constants, templates, wrappers, suggest, feed, wisp

**Integrations (6):**
claude, gemini, opencode, copilot, plugin, doltserver

**Specialized (8):**
activity, keepalive, nudge, swarm, dog, deps, doctor, connection, krc

## Source Tree

See `source-tree-analysis.md` for detailed annotated directory structure.

## Development Workflow

### Build Process
```
Source Code (Go)
    ↓ go generate
Generated Code
    ↓ make build
Compiled Binary (gt)
    ↓ ldflags (version injection)
Versioned Binary
    ↓ codesign (macOS)
Signed Binary
```

### Testing Strategy
- **Unit Tests:** 268 files, package-level
- **Integration Tests:** Cross-package functionality
- **E2E Tests:** Docker-isolated full workflow
- **Coverage:** ~39% ratio

## Deployment Architecture

### Build Targets
- Linux (amd64, arm64)
- macOS (amd64, arm64)
- Windows (amd64)
- FreeBSD (amd64)

### Distribution
- GitHub Releases (primary)
- Homebrew (macOS/Linux)
- NPM (via wrapper package)

### CI/CD
- 10 GitHub Actions workflows
- GoReleaser for multi-platform builds
- Automated releases on tag push

## Testing Strategy

### Test Pyramid
```
    ┌────────┐
    │  E2E   │  Docker-isolated (Dockerfile.e2e)
    │ Tests  │
    └────────┘
  ┌────────────┐
  │Integration│  Cross-package scenarios
  │   Tests   │
  └────────────┘
┌────────────────┐
│   Unit Tests   │  268 test files
│  (per package) │  Package-level testing
└────────────────┘
```

## Security & Reliability

### Security Measures
- **File Locking:** Prevents race conditions
- **Code Signing:** macOS binaries signed
- **Isolated Sessions:** Agents run in isolated contexts
- **Git Integrity:** Leverages git's built-in integrity checks

### Reliability Features
- **Crash Recovery:** Git-backed state survives crashes
- **Rollback:** Git history allows state rollback
- **Health Checks:** Doctor command validates environment
- **Daemon Supervision:** Process monitoring and restart

## Performance Characteristics

### Optimizations
- **Compiled Binary:** Fast startup (<100ms)
- **Concurrent Operations:** Go goroutines for parallelism
- **Lazy Loading:** Components loaded on-demand
- **Efficient TUI:** Bubbletea's minimal re-renders

### Scalability
- **Agent Limit:** 20-30 concurrent agents (design target)
- **Hook Capacity:** Hundreds of git worktrees
- **Convoy Size:** Thousands of work items (SQLite)

## Architectural Principles

1. **Persistence First:** Critical state survives crashes
2. **Runtime Agnostic:** Core logic independent of AI provider
3. **Git as Database:** Leverage git for versioned, distributed storage
4. **Fail Gracefully:** Degrade functionality vs crash
5. **Loose Coupling:** Event-driven communication
6. **Convention Over Config:** Sensible defaults
7. **Modularity:** Clear package boundaries

## Future Architecture Considerations

### Potential Enhancements
- **Federation:** Multi-town coordination
- **Plugin System:** Third-party extensions
- **Remote Agents:** Distributed agent execution
- **Alternative Storage:** Options beyond git worktrees
- **Web UI:** Full web-based interface
- **Metrics:** Opt-in telemetry and observability

## References

- **Source Tree:** `source-tree-analysis.md`
- **Technology Stack:** `technology-stack.md`
- **Architecture Patterns:** `architecture-patterns.md`
- **Integration:** `integration-architecture.md`
- **Development:** `development-guide.md`
- **Deployment:** `deployment-guide.md`
