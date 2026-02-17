# Gas Town - Project Overview

**Generated:** 2026-02-17
**Repository:** github.com/steveyegge/gastown
**License:** MIT

## What is Gas Town?

Gas Town is a multi-agent orchestration system for Claude Code and other AI coding assistants. It enables reliable coordination of multiple AI agents working on different tasks through git-backed persistent storage and structured workflow automation.

## Problem Statement

| Challenge | Gas Town Solution |
|-----------|-------------------|
| Agents lose context on restart | Work persists in git-backed hooks |
| Manual agent coordination | Built-in mailboxes, identities, and handoffs |
| 4-10 agents become chaotic | Scale comfortably to 20-30 agents |
| Work state lost in agent memory | Work state stored in persistent hooks |

## Project Type & Structure

**Repository Type:** Monolith with NPM distribution wrapper
**Primary Language:** Go 1.24.2
**Architecture:** CLI tool with daemon processes

### Project Parts

#### Part 1: gastown-cli (Primary)
- **Type:** CLI Tool with Backend Components
- **Language:** Go 1.24.2
- **Size:** 682 Go files across 58 packages
- **Purpose:** Core orchestration system

#### Part 2: npm-wrapper (Distribution)
- **Type:** NPM Package (Distribution Wrapper)
- **Language:** JavaScript (Node.js 14+)
- **Size:** ~100 lines
- **Purpose:** Cross-platform distribution via NPM

## Technology Stack Summary

### Core Technologies
| Technology | Purpose |
|-----------|---------|
| **Go 1.24.2** | Core implementation language |
| **Cobra** | CLI framework (command routing) |
| **Bubbletea** | Terminal UI framework |
| **SQLite** | Convoy database (work tracking) |
| **Git Worktrees** | Persistent storage (hooks) |
| **GoReleaser** | Multi-platform build automation |

### Key Dependencies
- Lipgloss (terminal styling)
- Rod (browser automation for web dashboard)
- gofrs/flock (file locking)
- google/uuid (identifier generation)
- BurntSushi/toml (configuration parsing)

## Architecture Classification

**Primary Pattern:** Layered CLI with Event-Driven Coordination

**Key Patterns:**
1. **Command Pattern** - Cobra command tree
2. **MVU Pattern** - Bubbletea terminal UI
3. **Git-Backed Persistence** - Innovative "Propulsion Principle"
4. **Plugin Architecture** - Multi-runtime agent providers
5. **Event-Driven** - Mail protocol for inter-agent communication
6. **Workflow Automation** - TOML-based formula system
7. **Daemon Processes** - Background orchestration (mayor, deacon, witness, refinery)

## Quick Reference

### Tech Stack
- **Language:** Go 1.24.2, JavaScript (Node.js 14+)
- **CLI:** Cobra (50+ commands)
- **TUI:** Bubbletea (Elm architecture)
- **Database:** SQLite (embedded)
- **Persistence:** Git worktrees
- **Build:** Make + GoReleaser
- **CI/CD:** GitHub Actions (10 workflows)

### Entry Points
- **Go Binary:** `cmd/gt/main.go`
- **NPM Wrapper:** `npm-package/bin/gt.js`

### Architecture Type
- **Pattern:** Multi-layered CLI
- **Communication:** Event-driven (mail protocol)
- **Persistence:** Git-backed (worktrees)
- **Distribution:** Multi-platform (6 targets)

## Core Concepts

### The Mayor 🎩
Your primary AI coordinator. Start here - tell the Mayor what you want to accomplish.

### Town 🏘️
Your workspace directory (e.g., `~/gt/`) containing all projects, agents, and configuration.

### Rigs 🏗️
Project containers. Each rig wraps a git repository and manages its associated agents.

### Crew Members 👤
Your personal workspace within a rig where you do hands-on work.

### Polecats 🦨
Worker agents with persistent identity but ephemeral sessions.

### Hooks 🪝
Git worktree-based persistent storage. Survives crashes and restarts.

### Convoys 🚚
Work tracking units that bundle multiple beads (tasks) for agents.

### Beads 📿
Git-backed issue tracking integration. Atomic work items with format: `gt-abc12`

### Formulas ⚙️
TOML-defined workflows for repeatable processes (33 built-in formulas).

## Key Features

✓ **Multi-Agent Orchestration** - Coordinate 20-30 AI agents
✓ **Persistent Work State** - Git-backed storage survives crashes
✓ **Multi-Runtime Support** - Claude, Codex, Cursor, Gemini, etc.
✓ **Workflow Automation** - 33 formula definitions for common tasks
✓ **Web Dashboard** - Real-time monitoring
✓ **Mail Protocol** - Inter-agent messaging
✓ **Convoy Tracking** - Work progress visualization
✓ **Cross-Platform** - Linux, macOS, Windows, FreeBSD

## Project Statistics

| Metric | Value |
|--------|-------|
| **Go Source Files** | 682 files |
| **Internal Packages** | 58 packages |
| **Test Files** | 268 files (39% coverage) |
| **CLI Commands** | 50+ commands |
| **Documentation Files** | 42+ markdown files |
| **Formulas** | 33 TOML workflows |
| **GitHub Workflows** | 10 CI/CD pipelines |
| **Supported Platforms** | 6 platform/arch combinations |

## Getting Started

### Installation

**Homebrew (macOS/Linux):**
```bash
brew install gastown
```

**NPM:**
```bash
npm install -g @gastown/gt
```

**From Source:**
```bash
git clone https://github.com/steveyegge/gastown.git
cd gastown
make install
```

### Quick Start

```bash
# Create workspace
gt install ~/gt --git
cd ~/gt

# Add project
gt rig add myproject https://github.com/you/repo.git

# Create crew workspace
gt crew add yourname --rig myproject
cd myproject/crew/yourname

# Start the Mayor
gt mayor attach
```

## Documentation Structure

### Getting Started
- **README.md** - Project introduction
- **INSTALLING.md** - Installation guide
- **CONTRIBUTING.md** - Contribution guidelines

### Architecture
- **architecture-gastown-cli.md** - Main architecture
- **architecture-npm-wrapper.md** - NPM wrapper architecture
- **architecture-patterns.md** - Pattern catalog
- **integration-architecture.md** - Multi-part integration
- **design/** - 17 design documents

### Concepts
- **concepts/** - 6 core concept documents
  - convoy.md
  - identity.md
  - molecules.md
  - polecat-lifecycle.md
  - propulsion-principle.md
  - integration-branches.md

### Development
- **development-guide.md** - Development workflow
- **deployment-guide.md** - Deployment process
- **technology-stack.md** - Tech stack details
- **source-tree-analysis.md** - Annotated source tree

### Analysis
- **comprehensive-analysis-gastown-cli.md** - Complete analysis
- **project-overview.md** (this file) - Project summary
- **component-inventory.md** - Component catalog

### Reference
- **reference.md** - Command reference
- **glossary.md** - Terminology
- **existing-documentation-inventory.md** - Doc index

## Links to Key Documents

### Architecture & Design
- [Main Architecture (CLI)](./architecture-gastown-cli.md)
- [NPM Wrapper Architecture](./architecture-npm-wrapper.md)
- [Architecture Patterns](./architecture-patterns.md)
- [Integration Architecture](./integration-architecture.md)
- [Technology Stack](./technology-stack.md)

### Development
- [Development Guide](./development-guide.md)
- [Deployment Guide](./deployment-guide.md)
- [Source Tree Analysis](./source-tree-analysis.md)

### Analysis
- [Comprehensive Analysis](./comprehensive-analysis-gastown-cli.md)
- [Component Inventory](./component-inventory.md)

### Original Documentation
- [Project README](../README.md)
- [HOOKS Documentation](./HOOKS.md)
- [Glossary](./glossary.md)
- [Reference](./reference.md)

## Development Workflow

### Prerequisites
- Go 1.24.2+
- Git 2.25+
- beads (bd) 0.44.0+
- sqlite3
- make

### Build Commands
```bash
make build              # Build binary
make install            # Install to ~/.local/bin
make test               # Run tests
make test-e2e-container # Run E2E tests (Docker)
make clean              # Clean artifacts
```

### Testing
- **Unit Tests:** 268 test files
- **Integration Tests:** Cross-package scenarios
- **E2E Tests:** Docker-isolated workflows

## Deployment

### Platforms
- Linux (amd64, arm64)
- macOS (amd64, arm64)
- Windows (amd64)
- FreeBSD (amd64)

### Distribution Channels
- **GitHub Releases** - Direct binary download
- **Homebrew** - `brew install gastown`
- **NPM** - `npm install -g @gastown/gt`

### CI/CD
- GitHub Actions (10 workflows)
- GoReleaser for multi-platform builds
- Automated releases on git tag

## Repository Information

**URL:** https://github.com/steveyegge/gastown
**License:** MIT
**Author:** Steve Yegge
**Primary Language:** Go (80%+), JavaScript (<5%), Others

## Contributing

See [CONTRIBUTING.md](../CONTRIBUTING.md) for contribution guidelines.

## Support & Community

- **Issues:** [GitHub Issues](https://github.com/steveyegge/gastown/issues)
- **Security:** See [SECURITY.md](../SECURITY.md)
- **Code of Conduct:** See [CODE_OF_CONDUCT.md](../CODE_OF_CONDUCT.md)

## Version Information

**Current Version:** 0.7.0
**Go Version:** 1.24.2
**NPM Version:** 0.7.0

## Next Steps

1. **For Users:** Read [README.md](../README.md) and [INSTALLING.md](./INSTALLING.md)
2. **For Developers:** Read [Development Guide](./development-guide.md)
3. **For Architects:** Read [Architecture Documentation](./architecture-gastown-cli.md)
4. **For Deep Dive:** Read [Comprehensive Analysis](./comprehensive-analysis-gastown-cli.md)
