---
project_name: 'gastown'
user_name: 'Jonathan'
date: '2026-02-17'
sections_completed: ['technology_stack', 'language_rules', 'framework_rules', 'testing_rules', 'code_quality_rules', 'workflow_rules', 'critical_rules']
existing_patterns_found: 15
status: complete
rule_count: 85
optimized_for_llm: true
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

**Language & Runtime:**
- Go 1.24.2 (primary language, strict requirement)
- Node.js 14+ (NPM wrapper distribution only)

**CLI & UI Frameworks:**
- Cobra v1.10.2 (CLI command framework)
- Bubbletea v1.3.10 (Terminal UI with MVU/Elm architecture)
- Lipgloss v1.1.1, Glamour v0.10.0 (terminal styling and markdown)

**Core Dependencies:**
- Rod v0.116.2 (browser automation via Chrome DevTools Protocol)
- gofrs/flock v0.13.0 (cross-platform file locking)
- google/uuid v1.6.0 (UUID generation)
- BurntSushi/toml v1.6.0 (TOML parsing)

**Persistence:**
- Git Worktrees (primary persistence via "Propulsion Principle")
- SQLite (embedded, no external deps)

**Build & Distribution:**
- Make (build orchestration with ldflags)
- GoReleaser v2 (multi-platform builds)
- golangci-lint v2.9.0 (**PINNED** - do not upgrade without testing)

**External Tools Required:**
- beads (bd) 0.44.0+ (formula system support)
- Git 2.25+ (worktree support required)
- tmux 3.0+ (optional, for multi-session)

---

## Critical Implementation Rules

### Language-Specific Rules (Go)

**Build & Version:**
- ❌ NEVER use `go build` directly - Always use `make build` (injects version via ldflags)
- ❌ NEVER use `go install` - Always use `make install` (removes stale binaries)
- ✅ Version flags required: Version, Commit, BuildTime, BuiltProperly

**Package Organization:**
- All implementation code in `internal/` (58 packages)
- Commands in `internal/cmd/`, domain logic in domain packages
- Use `go mod download` for dependencies

**Code Standards:**
- Must pass `gofmt` and `go vet`
- File naming: snake_case (e.g., `agent_state.go`)
- Type naming: PascalCase (exported), camelCase (unexported)
- Test files: `*_test.go` alongside source

**Error Handling:**
- No panic in library code - return errors explicitly
- Wrap errors with context using standard patterns

**Concurrency:**
- Use `gofrs/flock` for concurrent file access
- Serialize git worktree operations (not thread-safe)

**macOS:**
- Binaries automatically codesigned via Make
- Do not skip codesigning (required for modern macOS)

### Framework-Specific Rules

**Cobra (CLI Framework):**
- Root command: `gt` with 50+ subcommands organized by domain
- Commands in `internal/cmd/<command>.go`
- Use cobra.Command with RunE for error handling
- Maintain shell completion generation
- Flag-based configuration at each level

**Bubbletea (TUI/MVU Pattern) - CRITICAL:**
- ✅ Model: Immutable state only
- ✅ Update: Pure functions (state + message → new state)
- ✅ View: Rendering logic returns strings
- ❌ NEVER mutate model directly - always return new model
- Message flow: Input → Message → Update → Model → View → Output
- Used for: dashboards, progress displays, interactive prompts

**Git-Backed Persistence ("Propulsion Principle") - UNIQUE PATTERN:**
- Git worktrees are the primary persistence mechanism
- Each agent = dedicated git worktree in `.gt/hooks/agent-<id>/`
- State committed to git for crash survival
- ✅ All critical state must persist across crashes
- ✅ Serialize git operations (not thread-safe)
- ❌ NEVER assume in-memory state persists
- ❌ NEVER perform concurrent git worktree operations

### Testing Rules

**Test Organization:**
- Test files: `*_test.go` alongside source (268 test files)
- Integration tests: Separate files with integration build tag
- Current coverage: ~39% (268/682 files)

**E2E Testing - CRITICAL:**
- ❌ NEVER run E2E tests locally
- ✅ ONLY use: `make test-e2e-container`
- ✅ Tests run in isolated Docker container only
- Uses: `docker build -f Dockerfile.e2e -t gastown-test .`

**Test Execution:**
- Unit tests: `make test` or `go test ./...`
- Specific package: `go test ./internal/<package>`
- Coverage: `go test -cover ./...`
- All new functionality requires tests

**CI/CD:**
- 10 GitHub Actions workflows (ci, e2e, integration, windows-ci, etc.)
- golangci-lint v2.9.0 runs automatically
- Tests must pass before merge

### Code Quality & Style Rules

**Code Standards:**
- Must pass `gofmt` and `go vet`
- golangci-lint v2.9.0 in CI (pinned - do not upgrade without testing)
- Keep functions focused and small
- Comment non-obvious logic

**Naming Conventions:**
- Files: snake_case (`agent_state.go`, `convoy_lifecycle.go`)
- Tests: `*_test.go` (`agent_state_test.go`)
- Exported types: PascalCase (`AgentProvider`)
- Unexported types: camelCase
- Variables: camelCase

**File Organization:**
- Entry point: `cmd/gt/main.go` only
- All implementation: `internal/` (58 packages by domain)
- Commands: `internal/cmd/<command>.go`
- Domain packages: `internal/<domain>/`

**Documentation:**
- Public APIs require comments
- Non-obvious logic needs inline comments
- Keep comments concise and accurate

### Development Workflow Rules

**Branch Naming - CRITICAL:**
- ❌ NEVER create PRs from fork's `main` branch
- ✅ Always use dedicated branch per PR
- Prefixes: `fix/*`, `feat/*`, `refactor/*`, `docs/*`
- Reason: PRs from main accumulate all commits, prevents multiple PRs

**Commit Messages (Conventional Commits):**
- Format: `feat:`, `fix:`, `docs:`, `test:`, `chore:`, `refactor:`
- Present tense ("Add" not "Added")
- First line under 72 characters
- Reference issues: `Fix timeout bug (gt-xxx)`

**PR Requirements:**
- Tests pass: `make test`
- E2E pass: `make test-e2e-container` (if touching core)
- Build succeeds: `make build`
- Linting automatic in CI

**Installation:**
- ❌ NEVER use `go install` (creates stale binaries)
- ✅ Always use `make install` (removes stale from ~/go/bin)
- Update check enforced (override: `SKIP_UPDATE_CHECK=1`)

### Critical Don't-Miss Rules

**Anti-Patterns - Build & Installation:**
- ❌ `go build ./cmd/gt` - Missing version info
- ❌ `go install` - Creates stale binaries
- ❌ Upgrading golangci-lint - Version pinned at v2.9.0
- ✅ Always use `make build` and `make install`

**Anti-Patterns - Git Operations:**
- ❌ Concurrent git worktree operations - Not thread-safe
- ❌ Assuming in-memory state persists - Use git-backed hooks
- ❌ Skipping commits in worktrees - State won't survive crashes
- ✅ Serialize all git operations
- ✅ Commit critical state regularly

**Anti-Patterns - Bubbletea/MVU:**
- ❌ Mutating model directly - Violates MVU, causes bugs
- ❌ Side effects in Update function - Must be pure
- ✅ Return new model from Update
- ✅ Use Commands for side effects

**Edge Cases:**
- macOS: Automatic codesigning (don't skip)
- Windows: MinGW cross-compilation, CGO enabled
- FreeBSD: CGO disabled, static binary
- File locking varies by platform (use gofrs/flock)
- E2E tests ONLY work in Docker, not locally

**Performance Gotchas:**
- Git worktrees consume disk (full checkout each)
- Worktree creation is I/O intensive
- Cleanup stale worktrees to prevent bloat
- SQLite writes should be serialized
- Designed for max 20-30 concurrent agents

**Security:**
- Each agent isolated in own worktree
- Mail protocol for inter-agent communication
- Git commits provide audit trail
- No sudo required for normal operations

---

## Usage Guidelines

**For AI Agents:**
- Read this file before implementing any code
- Follow ALL rules exactly as documented
- When in doubt, prefer the more restrictive option
- Update this file if new patterns emerge

**For Humans:**
- Keep this file lean and focused on agent needs
- Update when technology stack changes
- Review quarterly for outdated rules
- Remove rules that become obvious over time

**Last Updated:** 2026-02-17
