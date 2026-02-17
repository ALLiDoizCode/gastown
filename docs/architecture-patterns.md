# Gas Town - Architecture Patterns

**Generated:** 2026-02-17
**Workflow Version:** 1.2.0

## Overall Architecture Classification

**Primary Pattern:** CLI Tool with Terminal UI and Git-Backed Persistence
**Secondary Patterns:** Event-Driven Coordination, Plugin Architecture, Daemon Processes

## Part 1: gastown-cli

### Architecture Pattern: Multi-Layered CLI with Persistent Orchestration

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interface Layer                     │
│  ┌────────────┐  ┌────────────┐  ┌──────────────────────┐  │
│  │ CLI (Cobra)│  │ TUI (Tea)  │  │ Web Dashboard (Rod)  │  │
│  └────────────┘  └────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                    Command Orchestration                     │
│  ┌─────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐  │
│  │  Mayor  │ │ Convoy │ │ Sling  │ │ Config │ │  Rig   │  │
│  └─────────┘ └────────┘ └────────┘ └────────┘ └────────┘  │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                    Core Domain Services                      │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌──────────┐ ┌────────┐ │
│  │ Agent  │ │ Convoy │ │ Hooks  │ │ Formula  │ │ Mail   │ │
│  │Provider│ │Manager │ │Registry│ │ Engine   │ │Protocol│ │
│  └────────┘ └────────┘ └────────┘ └──────────┘ └────────┘ │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                   Persistence & Runtime                      │
│  ┌────────────┐  ┌────────────┐  ┌──────────────────────┐  │
│  │Git Worktree│  │   SQLite   │  │  Agent Processes     │  │
│  │  (Hooks)   │  │  (Convoy)  │  │ (Claude/Codex/etc)   │  │
│  └────────────┘  └────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Key Architectural Patterns

#### 1. Command Pattern (CLI Layer)

**Implementation:** Cobra command tree

**Structure:**
- Root command: `gt`
- Command groups: workspace, agent ops, convoy, configuration
- Flag-based configuration at each level
- Shell completion generation

**Example:**
```
gt
├── install         # Workspace management
├── rig             # Project management
│   ├── add
│   ├── list
│   └── remove
├── crew            # Crew workspace
│   └── add
├── mayor           # AI coordinator
│   ├── attach
│   ├── start
│   └── detach
├── sling           # Work assignment
├── convoy          # Work tracking
│   ├── create
│   ├── list
│   ├── show
│   └── add
└── config          # Configuration
    ├── agent
    └── show
```

#### 2. Model-View-Update (MVU) Pattern (TUI Layer)

**Implementation:** Bubbletea framework (Elm architecture)

**Components:**
- **Model:** Application state (immutable)
- **Update:** State transitions based on messages
- **View:** Rendering logic (terminal output)

**Flow:**
1. User input → Message
2. Message → Update function
3. Update → New model + Commands
4. Model → View function
5. View → Terminal output

**Used in:** Dashboard, interactive prompts, progress displays

#### 3. Git-Backed Persistence ("Propulsion Principle")

**Unique architectural pattern:** Using git worktrees as persistent storage

**Concept:**
- Each agent gets a git worktree (hook)
- Work state committed to git
- Survives process crashes
- Version-controlled state
- Multi-agent coordination through git

**Benefits:**
- Persistence without external database
- Rollback capability
- Audit trail
- Distributed coordination

**Structure:**
```
.gt/
├── hooks/
│   ├── agent-abc123/     # Git worktree
│   │   ├── .git          # Points to main repo
│   │   ├── state.json
│   │   └── work/
│   └── agent-def456/     # Another worktree
└── .git/
```

#### 4. Plugin Architecture (Agent Providers)

**Pattern:** Abstract Factory + Strategy

**Abstraction Layer:**
```go
type AgentProvider interface {
    Start(config AgentConfig) (Session, error)
    Stop(sessionID string) error
    SendMessage(sessionID, message string) error
}
```

**Implementations:**
- ClaudeProvider
- CodexProvider
- CursorProvider
- GeminiProvider
- Custom agents via configuration

**Benefits:**
- Runtime-agnostic core logic
- Easy addition of new AI providers
- Testable through mocking
- User choice of AI backend

#### 5. Event-Driven Coordination (Mail Protocol)

**Pattern:** Message Queue + Mailbox

**Components:**
- **Sender:** Any agent or process
- **Mailbox:** Per-agent message queue
- **Receiver:** Agent checks mail on startup/poll
- **Protocol:** Structured message format

**Message Types:**
- Work assignment (sling)
- Status updates
- Handoffs
- Escalations

**Flow:**
```
Agent A → Mail → Agent B's Mailbox → Agent B reads → Action
```

#### 6. Workflow Automation (Formula System)

**Pattern:** Declarative Workflow Engine

**Components:**
- **Formula:** TOML-defined workflow
- **Variables:** User-provided inputs
- **Steps:** Ordered task definitions
- **Dependencies:** `needs` relationships

**Example Structure:**
```toml
[vars.version]
description = "Semantic version"
required = true

[[steps]]
id = "bump-version"
title = "Bump version"
description = "Run ./scripts/bump-version.sh {{version}}"

[[steps]]
id = "run-tests"
title = "Run tests"
needs = ["bump-version"]
```

**Integration:** Beads molecule system for tracking

#### 7. Daemon Architecture (Background Processes)

**Pattern:** Long-running service processes with supervision

**Daemons:**

| Daemon | Purpose | Lifecycle |
|--------|---------|-----------|
| **Mayor** | AI coordinator and primary interface | User-initiated, persistent session |
| **Deacon** | Background task execution and monitoring | Auto-started with town |
| **Witness** | Oversight and health monitoring | Auto-started with town |
| **Refinery** | Data processing and cleanup | Periodic execution |

**Communication:** Inter-process via mail protocol and shared state

### Data Flow Patterns

#### Convoy-Based Work Tracking

```
User Request
    ↓
Create Convoy (work bundle)
    ↓
Breakdown into Beads (atomic tasks)
    ↓
Sling Beads to Agents
    ↓
Agents execute in Hooks (persistent)
    ↓
Report completion to Convoy
    ↓
Convoy tracks overall progress
```

#### Agent Lifecycle

```
Spawn Request
    ↓
Create Hook (git worktree)
    ↓
Initialize Agent (provider-specific)
    ↓
Inject CLAUDE.md (role instructions)
    ↓
Check Mail (work assignment)
    ↓
Execute Work
    ↓
Commit State to Hook
    ↓
Report Completion
    ↓
Session Ends (hook persists)
```

## Part 2: npm-wrapper

### Architecture Pattern: Binary Distribution Wrapper

**Pattern:** Platform Detection + Binary Execution Wrapper

**Structure:**
```
npm install @gastown/gt
    ↓
Postinstall script runs
    ↓
Detect platform (darwin/linux/win32)
    ↓
Detect architecture (x64/arm64)
    ↓
Download/reference appropriate binary
    ↓
Set up executable wrapper (bin/gt.js)
    ↓
User runs 'gt' → wrapper invokes Go binary
```

**Benefits:**
- Familiar distribution channel for Node.js users
- Automatic platform detection
- Simple installation via npm
- Thin wrapper with minimal overhead

## Cross-Cutting Concerns

### Configuration Management

**Pattern:** Layered Configuration (Property Layers)

**Layers (priority order):**
1. Command-line flags (highest priority)
2. Environment variables
3. Project-level config (`settings/config.json`)
4. Town-level config
5. User-level config (`~/.config/gt/`)
6. Defaults (lowest priority)

**Configuration Types:**
- Runtime selection (Claude, Codex, etc.)
- Agent command and args
- Prompt modes
- Default behaviors

### Error Handling & Escalation

**Pattern:** Escalation Chain

**Levels:**
1. **Agent Self-Recovery:** Agent attempts to resolve issues
2. **Mayor Intervention:** Mayor provides guidance/resources
3. **Human Escalation:** User notified for decision
4. **Automatic Fallback:** Safe default action

### State Management

**Pattern:** Dual-State System

1. **Ephemeral State:** In-memory during agent session
2. **Persistent State:** Git-backed hooks
3. **Query State:** SQLite convoy database

**Synchronization:** Periodic commits from ephemeral to persistent

### Testing Strategy

**Pattern:** Multi-Layer Testing

1. **Unit Tests:** Go test for individual packages
2. **Integration Tests:** Cross-package integration
3. **E2E Tests:** Docker-isolated full workflow tests
4. **Manual Tests:** Real AI agent integration

## Architectural Principles

1. **Persistence First:** All critical state must survive crashes
2. **Runtime Agnostic:** Core logic independent of AI provider
3. **Git as Database:** Leverage git for versioned, distributed storage
4. **Fail Gracefully:** Degrade functionality rather than crash
5. **Coordination Through Messages:** Loose coupling via mail protocol
6. **Convention Over Configuration:** Sensible defaults, minimal setup
7. **Modularity:** Internal packages organized by domain

## Scalability Considerations

- **Agent Limit:** Designed for 20-30 concurrent agents
- **Hook Management:** Git worktrees scale to hundreds
- **Convoy Tracking:** SQLite handles thousands of work items
- **Message Queue:** In-memory mailboxes with file backup
- **Federation:** Future support for multi-town coordination
