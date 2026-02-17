# Gas Town - Project Structure

**Generated:** 2026-02-17
**Workflow Version:** 1.2.0

## Repository Classification

**Repository Type:** Monolith
**Project Name:** Gas Town (gastown)
**Primary Language:** Go 1.24.2

## Project Parts

### Part 1: gastown-cli (Primary)

**Type:** CLI Tool with Backend Components
**Root Path:** `/Users/jonathangreen/Documents/gastown`
**Project Type ID:** `cli`

**Key Technologies:**
- Go 1.24.2
- Cobra (CLI framework)
- Bubbletea (Terminal UI)
- Rod (Browser automation)
- SQLite (Convoy database)
- Git integration (worktrees, hooks)

**Purpose:** Multi-agent orchestration system for Claude Code with persistent work tracking

**Key Characteristics:**
- 40+ internal packages
- Web dashboard component
- Daemon processes (mayor, deacon)
- Git-backed persistent storage (hooks)
- Multi-runtime support (Claude, Codex, Cursor, etc.)

### Part 2: npm-wrapper (Secondary)

**Type:** Library/Distribution Package
**Root Path:** `/Users/jonathangreen/Documents/gastown/npm-package`
**Project Type ID:** `library`

**Key Technologies:**
- Node.js 14+
- NPM postinstall hooks

**Purpose:** NPM distribution wrapper for the Go binary

**Key Characteristics:**
- Postinstall script for binary setup
- Cross-platform support (darwin, linux, win32)
- Multi-architecture (x64, arm64)

## Integration Points

- **npm-wrapper → gastown-cli:** NPM package installs and wraps the Go binary
- **Build pipeline:** Go binary built separately, packaged by npm wrapper
