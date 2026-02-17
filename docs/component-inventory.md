# Gas Town - Component Inventory

**Generated:** 2026-02-17
**Total Components:** 58 internal packages
**Total Files:** 682 Go source files

## Component Categories

Components are organized into functional categories for clarity.

## 1. User Interface Components (5)

### 1.1 CLI Commands (internal/cmd)
**Purpose:** Command-line interface implementation
**Type:** Command handlers
**Key Files:** 50+ command implementations
**Dependencies:** Cobra, all internal packages
**Pattern:** Command pattern

**Major Commands:**
- install - Workspace initialization
- rig - Project management
- crew - Crew workspace
- mayor - AI coordinator
- sling - Work assignment
- convoy - Work tracking
- formula - Workflow automation
- config - Configuration management
- hooks - Hook management
- mail - Mail protocol

### 1.2 Terminal UI (internal/tui)
**Purpose:** Interactive terminal interfaces
**Type:** UI framework
**Technology:** Bubbletea (Elm architecture)
**Pattern:** Model-View-Update (MVU)

**Components:**
- Models - Application state
- Views - Rendering logic
- Update - State transitions
- Components - Reusable elements

### 1.3 UI Utilities (internal/ui)
**Purpose:** General UI helpers
**Type:** Utility functions
**Features:** Formatters, common elements

### 1.4 Style System (internal/style)
**Purpose:** Terminal styling and themes
**Type:** Styling library
**Technology:** Lipgloss
**Features:** Color schemes, text formatting, themes

### 1.5 Web Dashboard (internal/web)
**Purpose:** Web-based monitoring interface
**Type:** HTTP server
**Technology:** Rod (browser automation)
**Features:**
- Real-time agent status
- Convoy progress tracking
- Hook state visualization
- Configuration management

## 2. Orchestration Components (4)

### 2.1 Mayor (internal/mayor)
**Purpose:** AI coordinator daemon
**Type:** Coordinator
**Role:** Primary AI interface
**Responsibilities:**
- Task breakdown and delegation
- Context management
- Convoy orchestration
- Agent coordination

### 2.2 Convoy Manager (internal/convoy)
**Purpose:** Work tracking system
**Type:** Manager
**Database:** SQLite
**Features:**
- Convoy creation and management
- Progress tracking
- Bead assignment
- Status reporting

### 2.3 Formula Engine (internal/formula)
**Purpose:** Workflow automation
**Type:** Engine
**Format:** TOML workflows
**Features:**
- Variable substitution
- Step execution
- Dependency resolution
- 33 built-in formulas

### 2.4 Mail Protocol (internal/mail)
**Purpose:** Inter-agent communication
**Type:** Message queue
**Features:**
- Per-agent mailboxes
- Identity management
- Group communication
- Message routing

## 3. Agent Management Components (4)

### 3.1 Agent Provider (internal/agent)
**Purpose:** Agent lifecycle management
**Type:** Abstract factory
**Pattern:** Provider interface
**Features:**
- Multi-runtime support
- Session management
- Agent spawning
- Provider abstraction

### 3.2 Runtime Abstraction (internal/runtime)
**Purpose:** Runtime provider registry
**Type:** Registry
**Supported Runtimes:**
- Claude Code
- Codex
- Cursor
- Gemini
- Custom agents

### 3.3 Polecat Worker (internal/polecat)
**Purpose:** Worker agent implementation
**Type:** Agent
**Characteristics:**
- Ephemeral sessions
- Persistent identity
- Work execution

### 3.4 Crew Workspace (internal/crew)
**Purpose:** Human crew member workspace
**Type:** Workspace manager
**Features:**
- Workspace creation
- Integration with rig

## 4. Persistence & State Components (5)

### 4.1 Hooks Registry (internal/hooks)
**Purpose:** Git-backed persistence
**Type:** Registry
**Innovation:** Git worktrees as storage
**Features:**
- Hook creation/deletion
- Worktree management
- State commits
- Version control

### 4.2 State Management (internal/state)
**Purpose:** Application state coordination
**Type:** State manager
**Components:**
- Global state
- Per-agent state
- State serialization

### 4.3 Session Management (internal/session)
**Purpose:** Session lifecycle
**Type:** Manager
**Features:**
- Session creation
- Session recovery
- Crash recovery

### 4.4 Checkpoint System (internal/checkpoint)
**Purpose:** State snapshots
**Type:** Checkpoint
**Features:**
- State checkpoints
- Rollback capability

### 4.5 Lock Manager (internal/lock)
**Purpose:** Concurrency control
**Type:** Lock manager
**Technology:** gofrs/flock
**Features:**
- File locking
- Resource locks
- Race prevention

## 5. Communication Components (4)

### 5.1 Mail System (internal/mail)
**Purpose:** Inter-agent messaging
**Type:** Message queue
*(Detailed above in Orchestration)*

### 5.2 Event System (internal/events)
**Purpose:** Event-driven coordination
**Type:** Publisher-subscriber
**Features:**
- Event emission
- Event handlers
- Async processing

### 5.3 Message Queue (internal/mq)
**Purpose:** Message queue operations
**Type:** Queue
**Features:**
- Queue management
- Message persistence

### 5.4 Protocol Definitions (internal/protocol)
**Purpose:** Communication protocols
**Type:** Protocol definitions
**Features:**
- Message formats
- Protocol specs

## 6. Infrastructure Components (8)

### 6.1 Git Operations (internal/git)
**Purpose:** Git repository management
**Type:** Git wrapper
**Features:**
- Worktree operations
- Branch management
- Commit operations
- Remote sync

### 6.2 Configuration (internal/config)
**Purpose:** Configuration management
**Type:** Config loader
**Features:**
- Layered configuration
- Role definitions (7 TOML files)
- Runtime settings
- Property layers

### 6.3 Workspace Manager (internal/workspace)
**Purpose:** Town workspace operations
**Type:** Workspace manager
**Features:**
- Town initialization
- Workspace management

### 6.4 Rig Manager (internal/rig)
**Purpose:** Project container management
**Type:** Project manager
**Features:**
- Rig creation
- Git repository wrapping
- Per-rig configuration

### 6.5 Boot System (internal/boot)
**Purpose:** System bootstrap
**Type:** Initializer
**Features:**
- Initialization sequences
- Startup checks

### 6.6 Daemon Utilities (internal/daemon)
**Purpose:** Daemon management
**Type:** Daemon support
**Features:**
- Daemon lifecycle
- Process supervision

### 6.7 Tmux Integration (internal/tmux)
**Purpose:** Tmux session management
**Type:** Integration
**Features:**
- Session management
- Pane operations
- Window management

### 6.8 Shell Operations (internal/shell)
**Purpose:** Shell command execution
**Type:** Shell wrapper
**Features:**
- Command execution
- Shell detection

## 7. Daemon Process Components (4)

### 7.1 Mayor Daemon (internal/mayor)
**Purpose:** AI coordinator
*(Detailed above in Orchestration)*

### 7.2 Deacon Daemon (internal/deacon)
**Purpose:** Background task manager
**Type:** Daemon
**Features:**
- Periodic task execution
- Background monitoring

### 7.3 Witness Daemon (internal/witness)
**Purpose:** Oversight and monitoring
**Type:** Daemon
**Features:**
- Health checks
- Alerting
- Oversight

### 7.4 Refinery Daemon (internal/refinery)
**Purpose:** Data processing
**Type:** Daemon
**Features:**
- Data cleanup
- Normalization
- Garbage collection

## 8. Integration Components (6)

### 8.1 Claude Integration (internal/claude)
**Purpose:** Claude Code runtime
**Type:** Provider implementation
**Features:**
- Claude-specific operations
- Hook injection
- Session management

### 8.2 Gemini Integration (internal/gemini)
**Purpose:** Google Gemini runtime
**Type:** Provider implementation

### 8.3 OpenCode Integration (internal/opencode)
**Purpose:** OpenAI Codex integration
**Type:** Provider implementation

### 8.4 Copilot Integration (internal/copilot)
**Purpose:** GitHub Copilot integration
**Type:** Provider implementation

### 8.5 Plugin System (internal/plugin)
**Purpose:** Plugin architecture
**Type:** Plugin loader
**Features:**
- Plugin loading
- Plugin API
- Extension points

### 8.6 Beads Integration (internal/beads)
**Purpose:** Beads issue tracking
**Type:** Integration
**Features:**
- Bead ID handling
- Custom types
- Git-backed issues

## 9. Utility Components (10)

### 9.1 General Utilities (internal/util)
**Purpose:** Common helper functions
**Type:** Utilities
**Features:** General-purpose helpers

### 9.2 Style Utilities (internal/style)
*(Detailed above in User Interface)*

### 9.3 Logging (internal/townlog)
**Purpose:** Structured logging
**Type:** Logger
**Features:**
- Log levels
- Structured logging
- Formatting

### 9.4 Version Management (internal/version)
**Purpose:** Version information
**Type:** Version metadata
**Features:**
- Version info
- Build metadata

### 9.5 Constants (internal/constants)
**Purpose:** Application constants
**Type:** Constants
**Features:**
- Fixed values
- Configuration defaults

### 9.6 Templates (internal/templates)
**Purpose:** Template system
**Type:** Templates
**Features:**
- Role templates
- Configuration templates
- Agent templates (in templates/agents/)

### 9.7 Wrappers (internal/wrappers)
**Purpose:** External tool wrappers
**Type:** Wrappers
**Features:**
- Command wrappers
- API wrappers

### 9.8 Suggest Engine (internal/suggest)
**Purpose:** Suggestion and autocomplete
**Type:** Suggestion engine
**Features:**
- Autocomplete
- Recommendations

### 9.9 Feed System (internal/feed)
**Purpose:** Activity feed
**Type:** Feed
**Features:**
- Event stream
- Activity log display

### 9.10 Wisp Protocol (internal/wisp)
**Purpose:** Lightweight communication
**Type:** Protocol
**Features:**
- Fast messaging
- Compact protocol

## 10. Specialized Components (8)

### 10.1 Molecule System (internal/molecule)
**Purpose:** Formula instance tracking
**Type:** Tracker
**Note:** Possibly deprecated/experimental

### 10.2 Wasteland Manager (internal/wasteland)
**Purpose:** Orphaned work management
**Type:** Manager
**Features:**
- Unclaimed work
- Work recovery

### 10.3 Activity Tracker (internal/activity)
**Purpose:** Activity tracking
**Type:** Tracker
**Features:**
- User activity logs
- Agent activity logs

### 10.4 Keepalive (internal/keepalive)
**Purpose:** Connection keepalive
**Type:** Health check
**Features:**
- Connection keepalive
- Health pings

### 10.5 Nudge System (internal/nudge)
**Purpose:** Notification system
**Type:** Notifier
**Features:**
- User notifications
- Agent notifications

### 10.6 Swarm Coordination (internal/swarm)
**Purpose:** Multi-agent swarm
**Type:** Coordinator
**Features:**
- Swarm behavior
- Collective tasks

### 10.7 Dog Watchdog (internal/dog)
**Purpose:** Watchdog functionality
**Type:** Monitor
**Features:**
- Process monitoring
- Auto-restart

### 10.8 Dependency Manager (internal/deps)
**Purpose:** Dependency management
**Type:** Manager
**Features:**
- External dependencies
- Version checking

## 11. Diagnostic Components (2)

### 11.1 Doctor Tool (internal/doctor)
**Purpose:** Diagnostic and health checks
**Type:** Diagnostic
**Features:**
- Environment validation
- Configuration checks
- Health verification

### 11.2 Connection Manager (internal/connection)
**Purpose:** Connection management
**Type:** Manager
**Features:**
- Network connections
- Session connections

## 12. Experimental/Deprecated Components (2)

### 12.1 DoltServer (internal/doltserver)
**Purpose:** Dolt database server
**Type:** Database integration
**Status:** Experimental/Deprecated
**Note:** Dolt integration (may not be active)

### 12.2 KRC (internal/krc)
**Purpose:** Unknown
**Type:** Unknown
**Status:** Unidentified component

## Reusable Components

### High Reusability
- **util** - General utilities (used by all)
- **config** - Configuration (used by all)
- **townlog** - Logging (used by all)
- **git** - Git operations (hooks, rig, workspace)
- **state** - State management (agent, session, convoy)

### Medium Reusability
- **style** - Styling (tui, web, cli)
- **events** - Events (mail, convoy, agent)
- **lock** - Locking (hooks, convoy, state)
- **templates** - Templates (config, agent)

### Low Reusability (Specialized)
- **mayor, deacon, witness, refinery** - Daemon-specific
- **claude, gemini, opencode, copilot** - Runtime-specific
- **wasteland, swarm, dog** - Specialized functionality

## Component Dependencies

### Most Depended Upon
1. **config** - Configuration (58 components depend on it)
2. **util** - Utilities (50+ components)
3. **townlog** - Logging (50+ components)
4. **git** - Git operations (hooks, rig, workspace, convoy)
5. **state** - State management (agent, convoy, session)

### Least Dependencies (Leaf Components)
1. **constants** - Pure constants
2. **version** - Version metadata
3. **style** - Styling definitions
4. **templates** - Template files

## Component Statistics

| Category | Package Count |
|----------|---------------|
| User Interface | 5 |
| Orchestration | 4 |
| Agent Management | 4 |
| Persistence & State | 5 |
| Communication | 4 |
| Infrastructure | 8 |
| Daemons | 4 |
| Integrations | 6 |
| Utilities | 10 |
| Specialized | 8 |
| Diagnostic | 2 |
| Experimental | 2 |
| **Total** | **58** |

## Entry Point Components

### Primary Entry
- **cmd/gt/main.go** - Application entry point
  - Delegates to: internal/cmd.Execute()

### Command Router
- **internal/cmd** - Cobra command tree
  - Routes to: All 58 internal packages

### Daemon Entries
- **internal/mayor** - Mayor daemon entry
- **internal/deacon** - Deacon daemon entry
- **internal/witness** - Witness daemon entry
- **internal/refinery** - Refinery daemon entry

## Critical Path Components

**For Basic Operation:**
1. cmd → internal/cmd → config → workspace/rig → agent → hooks

**For Work Tracking:**
1. convoy → beads → mail → agent → hooks

**For Workflow Automation:**
1. formula → convoy → mail → agent → hooks

**For Persistence:**
1. hooks → git → state → lock

## Component Maturity

### Mature (Stable, Well-Tested)
- cmd, agent, hooks, convoy, config, git, state

### Active Development
- mayor, formula, mail, web, swarm

### Experimental
- molecule, doltserver, krc, wasteland

## Future Components

**Potential Additions:**
- **federation** - Multi-town coordination
- **metrics** - Observability and telemetry
- **cache** - Caching layer
- **search** - Full-text search
- **backup** - Backup and restore

## References

- **Architecture:** `architecture-gastown-cli.md`
- **Source Tree:** `source-tree-analysis.md`
- **Comprehensive Analysis:** `comprehensive-analysis-gastown-cli.md`
