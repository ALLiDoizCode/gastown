---
stepsCompleted: ['step-01-validate-prerequisites', 'step-02-design-epics', 'step-03-create-stories']
inputDocuments:
  - /Users/jonathangreen/Documents/gastown/_bmad-output/planning-artifacts/prd/index.md
  - /Users/jonathangreen/Documents/gastown/_bmad-output/planning-artifacts/architecture/index.md
epicsCompleted: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
---

# gastown - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for gastown Crosstown Integration, decomposing the requirements from the PRD and Architecture documents into implementable stories.

## Requirements Inventory

### Functional Requirements

**FR1:** Mayor can connect to crosstown relay via NIP-01 WebSocket protocol

**FR2:** Mayor can subscribe to specific Nostr event kinds with subscription filters (kind:5xxx DVM jobs, kind:6xxx DVM results)

**FR3:** Mayor can receive TOON-encoded events from relay via WebSocket

**FR4:** Mayor can reconnect to relay automatically on connection loss with exponential backoff retry logic

**FR5:** Mayor can display relay connection status in real-time

**FR6:** Operator can configure wallet with encrypted private key via CLI commands and environment variables

**FR7:** Mayor can perform SPSP handshake with crosstown connector automatically on startup

**FR8:** Mayor can negotiate settlement chain (Base Sepolia testnet) during SPSP handshake

**FR9:** Mayor can open payment channel with M2M token deposit on negotiated settlement chain

**FR10:** Mayor can persist payment channel state (channel ID, balance, settlement address) in git worktree

**FR11:** Mayor can monitor payment channel balance and display warnings when balance drops below 10 M2M tokens (configurable threshold)

**FR12:** Mayor can close payment channel and withdraw remaining funds

**FR13:** Mayor can reconstruct payment channel state from on-chain events if git state is lost

**FR14:** Mayor can validate relay pricing (basePricePerByte) during SPSP handshake and warn if exceeds 100 satoshis/byte (configurable threshold)

**FR15:** Mayor can construct kind:5xxx DVM job request events with TOON encoding

**FR16:** Mayor can send ILP PREPARE packets with TOON-encoded events as packet data payload

**FR17:** Mayor can handle ILP FULFILL packets (job posted successfully) and ILP REJECT packets (payment failed)

**FR18:** Mayor can store job correlation mappings (job_id → callback_skill) in git worktree for result routing

**FR19:** Mayor can subscribe to kind:6xxx result events filtered by posted job IDs

**FR20:** Mayor can route received kind:6xxx results to appropriate callback skills based on job correlation

**FR21:** Mayor can execute sequential job chains (Job A completion triggers Job B posting)

**FR22:** Mayor can resume in-flight job chains after crash using git-backed correlation state

**FR23:** Mayor can handle job timeouts and fail gracefully if no result received within configurable time

**FR24:** Polecat can execute assigned jobs in dedicated git worktrees

**FR25:** Mayor can construct kind:6xxx result events with job correlation for completed work

**FR26:** Mayor can publish result events via ILP PREPARE packets and earn M2M token payments

**FR27:** Mayor can pass raw TOON byte arrays to Claude Code with zero parsing in Go networking layer

**FR28:** Claude Code can interpret TOON structure directly using LLM-native processing (pattern recognition on binary byte sequences without explicit schema definitions, leveraging TOON's structural regularity to extract fields with fewer tokens than JSON parsing)

**FR29:** Claude Code can extract event kind field from TOON payload for routing decisions

**FR30:** Claude Code can route raw TOON payloads to appropriate skills based on extracted kind

**FR31:** Skills can handle kind:5xxx DVM job requests with raw TOON payloads

**FR32:** Skills can handle kind:6xxx DVM result events with raw TOON payloads

**FR33:** Operator can configure wallet settings via environment variables (address, encrypted private key, RPC URL)

**FR34:** Operator can configure crosstown settings (relay URL, connector endpoint) via environment variables, config file (~/.gt/crosstown.toml), or CLI commands

**FR35:** Mayor can enforce private key encryption at rest and refuse to start if keys unencrypted unless override flag explicitly set

**FR36:** Mayor can prompt for encryption passphrase at runtime without storing or logging passphrase

**FR37:** Mayor can apply configuration from multiple sources with defined priority (CLI flags > env vars > config file > defaults)

**FR38:** Operator can override security defaults with explicit warning flags (GASTOWN_ALLOW_UNENCRYPTED_KEYS)

**FR39:** Mayor can display Bubbletea dashboard with crosstown network panel showing connection status, channel balance, subscribed events, and recent activity

**FR40:** Mayor can log every ILP payment to git audit trail with commit per payment including job ID, amount, bytes, and status

**FR41:** Mayor can log structured payment data to append-only log file (payment_log.jsonl) with rotation

**FR42:** Mayor can export payment history to CSV and JSON formats with date filtering

**FR43:** Mayor can display payment statistics (total payments, total M2M spent, average cost per event)

**FR44:** Mayor can log payment channel state changes (open/close events) with on-chain transaction hashes

**FR45:** Mayor can display active network (Base Sepolia testnet vs mainnet) prominently to prevent deployment confusion

**FR46:** Operator can run Mayor in interactive mode (gt mayor attach) with real-time Bubbletea dashboard

**FR47:** Operator can manage wallet via CLI commands (configure, check balance, export encrypted key)

**FR48:** Operator can manage payment channels via CLI commands (open, check status, close, list all channels)

**FR49:** Operator can manage relay connection via CLI commands (subscribe, disconnect, check status)

**FR50:** Operator can export payment history via CLI command (gt mayor export-payments) with format and date options

**FR51:** All CLI commands can output in multiple formats (human-readable text, JSON, CSV) via --format flag

**FR52:** All CLI commands can run in non-interactive mode with --yes flag to skip confirmations for automation

**FR53:** Operator can generate shell completion scripts for bash, zsh, fish, and PowerShell

**FR54:** Operator can automate crosstown operations via shell scripts using environment variable configuration and non-interactive flags

**FR55:** Developer can create skill files in .claude/skills/ directory with zero configuration

**FR56:** Claude Code can automatically discover skills from .claude/skills/ and route events based on skill-declared supported kinds

**FR57:** Skills can receive raw TOON payloads for processing without requiring JSON conversion

**FR58:** Developer can extend Mayor to handle new Nostr event kinds by adding skill files with zero changes to Mayor core code

### NonFunctional Requirements

**NFR-P1:** ILP payment flow (PREPARE → FULFILL) must complete in less than 500ms under normal network conditions to avoid blocking agent work

**NFR-P2:** Extracting event `kind` field from TOON payload must complete in less than 1ms per event to support high-throughput event processing

**NFR-P3:** Mayor's NIP-01 WebSocket subscription must handle at least 10 events per second without dropping messages or degrading dashboard responsiveness

**NFR-P4:** Bubbletea dashboard must reflect state changes (connection status, channel balance, new events) within 100ms of occurrence for real-time monitoring

**NFR-P5:** SPSP handshake and payment channel opening must complete within 30 seconds to avoid operator frustration during setup

**NFR-P6:** Payment audit commits to git worktree must not block ILP FULFILL responses (asynchronous logging acceptable)

**NFR-S1:** All wallet private keys MUST be encrypted at rest using AES-256-GCM or equivalent industry-standard encryption. Mayor must refuse to start if private keys are unencrypted unless explicit override flag (GASTOWN_ALLOW_UNENCRYPTED_KEYS=true) is set with warning displayed

**NFR-S2:** Encryption passphrase for private keys must be prompted at runtime (not stored in env vars or config files), not logged to any file or console output, stored only in memory during Mayor session, and cleared from memory on Mayor shutdown

**NFR-S3:** BTP WebSocket connections to crosstown connector must use TLS (wss://) for encrypted transport. Mayor must validate connector TLS certificates

**NFR-S4:** Mayor must enforce maximum balance limit per payment channel (default: 100 M2M tokens for testnet, configurable via GASTOWN_MAX_CHANNEL_BALANCE_M2M). Channel opening must fail if deposit exceeds limit

**NFR-S5:** Payment channel state files in git worktree must be created with Unix file permissions 0600 (owner read/write only) to prevent unauthorized access

**NFR-S6:** Payment audit trail (git commits) must be tamper-evident. Git commit history provides cryptographic verification of audit log integrity

**NFR-S7:** Mayor must validate basePricePerByte received during SPSP handshake against expected range. Warn operator if price exceeds configurable threshold (default: >100 units per byte)

**NFR-S8:** Mayor dashboard must prominently display active network (Base Sepolia testnet vs mainnet) to prevent accidental mainnet deployment. Mainnet requires explicit --enable-mainnet flag

**NFR-R1:** All critical state (job correlation mappings, sequential job chain state, subscription filters) must be committed to git worktree and survive Mayor process crashes or system restarts. Mayor must resume from git state on restart with zero data loss

**NFR-R2:** Payment channel state (channel IDs, balances, settlement addresses) must survive crashes. State is stored both in git worktree (local cache) and on-chain (source of truth). Mayor must reconstruct state from on-chain events if git state is lost

**NFR-R3:** Mayor must automatically reconnect to crosstown relay on connection loss with exponential backoff retry logic (initial: 1s, max: 30s, jitter to prevent thundering herd)

**NFR-R4:** Failed ILP PREPARE packets must be retried with exponential backoff (max 3 retries) before failing job posting. Transient network errors must not cause permanent job loss

**NFR-R5:** If no kind:6xxx result received within configurable timeout (default: 300 seconds), job must be marked as failed with timeout error. Mayor must not block indefinitely waiting for results

**NFR-R6:** All git worktree operations must be serialized (not concurrent) to prevent race conditions. gastown's existing git worktree locking mechanism must be respected

**NFR-R7:** If crosstown relay is unavailable, Mayor must continue operating in local-only mode (Polecat orchestration continues, network job distribution disabled). Dashboard must display relay disconnection status

**NFR-R8:** Git worktrees containing payment channel state should be backed up to remote git repositories (GitHub/GitLab) to provide disaster recovery capability

**NFR-I1:** Mayor's relay client must fully comply with NIP-01 (Nostr Basic Protocol) specification for REQ/EVENT/EOSE/CLOSE message handling

**NFR-I2:** Mayor's BTP client must comply with Interledger BTP (Bilateral Transfer Protocol) specification for packet framing and WebSocket transport

**NFR-I3:** ILP PREPARE/FULFILL/REJECT packets must comply with ILPv4 (RFC 0027) specification for packet structure and OER encoding

**NFR-I4:** Mayor must correctly encode and decode TOON-formatted Nostr events following the TOON specification (https://github.com/nicholasgasior/toon) with LLM prompting best practices (https://toonformat.dev/guide/llm-prompts.html)

**NFR-I5:** New gt crosstown command group must integrate with existing gastown Cobra command structure following established patterns (RunE for error handling, PreRun for validation)

**NFR-I6:** Crosstown dashboard panel must integrate with existing Mayor TUI following Bubbletea MVU (Model-View-Update) pattern. State must be immutable, updates must be pure functions

**NFR-I7:** Payment channel state and job correlation must persist in Mayor's existing git worktree following gastown's Propulsion Principle. No new persistence mechanism introduced

**NFR-I8:** TOON event routing must integrate with Claude Code's existing skills system. Go code passes raw TOON bytes to Claude Code via skill invocation interface with zero changes to skill API

**NFR-I9:** On-chain payment channel operations must support Base Sepolia testnet (EVM chain ID 84532) using standard Web3 RPC endpoints (https://sepolia.base.org)

**NFR-I10:** Crosstown configuration must extend gastown's existing config system (env vars, TOML config file) without breaking existing configuration patterns

**NFR-SC1:** Mayor must support gastown's documented constraint of max 20-30 concurrent Polecats without performance degradation or resource exhaustion

**NFR-SC2:** Mayor must support at least 5 concurrent open payment channels (to different relays/connectors) without state management issues

**NFR-SC3:** Mayor must handle subscriptions to at least 10 event kinds simultaneously (kind:5xxx, kind:6xxx, future kind:1, kind:1059, etc.) without filter management complexity

**NFR-SC4:** Git-backed job correlation storage must support at least 1000 active job mappings (job_id → callback_skill) without git repository bloat or performance issues

**NFR-SC5:** Payment log file (payment_log.jsonl) must rotate daily and retain 30 days of history without manual intervention. Log rotation must not impact Mayor availability

### Additional Requirements

**Brownfield Integration Context:**
- This is a brownfield integration into existing gastown v0.7.0 architecture, not a greenfield project
- Must preserve all existing commands and workflows
- Must maintain backward compatibility
- Must respect existing git-backed persistence model
- Must adhere to existing build and test requirements

**Technology Stack Constraints:**
- Go 1.24.2 strict requirement (cannot upgrade)
- CGO enabled for most platforms (disabled for FreeBSD)
- Cobra v1.10.2 for command structure
- Bubbletea v1.3.10 with Elm/MVU architecture for TUI
- Lipgloss v1.1.1 for terminal styling
- Git worktrees as primary persistence ("Propulsion Principle")
- SQLite embedded database for convoy queries
- TOML configuration files

**Library Dependencies:**
- gorilla/websocket (latest stable) for WebSocket client
- net/http (stdlib) for HTTP client
- github.com/ALLiDoizCode/toon-go (fork) for TOON encoding/decoding
- golang.org/x/crypto/argon2 for key derivation
- github.com/golang/mock for test mocking

**Package Structure Requirements:**
- New packages to add: internal/ilp/, internal/nostr/, internal/crosstown/
- Modified packages: internal/cmd/ (add crosstown.go), internal/mayor/ (add dashboard crosstown panel)
- All implementation code must be in internal/ directory
- Entry point remains cmd/gt/main.go only

**Testing Infrastructure Requirements:**
- Unit tests: *_test.go alongside source with gomock for mocking
- Integration tests: Separate with build tags
- E2E tests: ONLY in Docker containers (make test-e2e-container)
- golangci-lint v2.9.0 (pinned)
- Minimum 50% code coverage required

**Build and Deployment Requirements:**
- Makefile-based build orchestration
- Version info injected via ldflags
- GoReleaser v2 for multi-platform releases
- Automatic macOS codesigning
- Docker Compose for E2E test environment

**Configuration System Requirements:**
- Support environment variables with GASTOWN_* prefix
- Support TOML config file (~/.gt/crosstown.toml)
- Support CLI flags
- Priority: CLI flags > env vars > config file > defaults

**Security Requirements:**
- Payment channel state files must use Unix permissions 0600
- Argon2id key derivation with parameters: Time=1, Memory=64MB, Threads=4, KeyLen=32
- Private key encryption at rest required unless explicit override
- Passphrase prompted at runtime, never logged or stored

**File Organization Requirements:**
- Payment channel state: One TOML file per channel in ~/.gt/hooks/mayor-{id}/crosstown/channels/
- Git commit per payment for audit trail
- Job correlation state stored in git worktree
- Payment logs in append-only JSONL format with daily rotation

**Development Patterns to Follow:**
- Command Pattern (Cobra CLI tree)
- MVU Architecture (Bubbletea)
- Plugin Architecture (agent providers)
- Event-Driven Coordination (mail protocol)
- Git-Backed Persistence (hooks)
- Immutable models, pure update functions for Bubbletea
- Message-driven state updates

### FR Coverage Map

**Epic 1:** Architecture package structure requirements (internal/ilp/, internal/nostr/, internal/crosstown/)
**Epic 2:** FR6, FR33, FR35, FR36, FR38, FR47 (wallet management)
**Epic 3:** FR1, FR2, FR3, FR4, FR5
**Epic 4:** FR7, FR8, FR9, FR10, FR11, FR12, FR13, FR14, FR48 (channel CLI)
**Epic 5:** FR15, FR16, FR17, FR18
**Epic 6:** FR24, FR25, FR26
**Epic 7:** FR19, FR20
**Epic 8:** FR27, FR28, FR29, FR30, FR31, FR32, FR55, FR56, FR57, FR58
**Epic 9:** FR21, FR22, FR23
**Epic 10:** FR39, FR40, FR41, FR42, FR43, FR44, FR45, FR46
**Epic 11:** FR34, FR37, FR47 (wallet CLI), FR48 (channel CLI), FR49, FR50, FR51, FR52, FR53, FR54
**Epic 12:** Testing and production readiness requirements

**Detailed FR Mapping:**

FR1 → Epic 3 - Connect to crosstown relay via NIP-01 WebSocket
FR2 → Epic 3 - Subscribe to specific Nostr event kinds
FR3 → Epic 3 - Receive TOON-encoded events from relay
FR4 → Epic 3 - Auto-reconnect on connection loss with backoff
FR5 → Epic 3 - Display relay connection status in real-time
FR6 → Epic 2 - Configure wallet with encrypted private key
FR7 → Epic 4 - Perform SPSP handshake with connector
FR8 → Epic 4 - Negotiate settlement chain during handshake
FR9 → Epic 4 - Open payment channel with M2M deposit
FR10 → Epic 4 - Persist payment channel state in git
FR11 → Epic 4 - Monitor channel balance with warnings
FR12 → Epic 4 - Close payment channel and withdraw funds
FR13 → Epic 4 - Reconstruct channel state from on-chain
FR14 → Epic 4 - Validate relay pricing during handshake
FR15 → Epic 5 - Construct kind:5xxx DVM job requests
FR16 → Epic 5 - Send ILP PREPARE packets with TOON payloads
FR17 → Epic 5 - Handle ILP FULFILL/REJECT packets
FR18 → Epic 5 - Store job correlation mappings in git
FR19 → Epic 7 - Subscribe to kind:6xxx result events
FR20 → Epic 7 - Route results to callback skills
FR21 → Epic 9 - Execute sequential job chains
FR22 → Epic 9 - Resume in-flight jobs after crash
FR23 → Epic 9 - Handle job timeouts gracefully
FR24 → Epic 6 - Polecat executes jobs in worktrees
FR25 → Epic 6 - Construct kind:6xxx result events
FR26 → Epic 6 - Publish results and earn M2M payments
FR27 → Epic 8 - Pass raw TOON bytes to Claude Code
FR28 → Epic 8 - LLM-native TOON processing
FR29 → Epic 8 - Extract event kind field from TOON
FR30 → Epic 8 - Route TOON payloads to skills
FR31 → Epic 8 - Skills handle kind:5xxx with TOON
FR32 → Epic 8 - Skills handle kind:6xxx with TOON
FR33 → Epic 2 - Configure wallet via environment variables
FR34 → Epic 11 - Configure crosstown settings (env/config/CLI)
FR35 → Epic 2 - Enforce private key encryption at rest
FR36 → Epic 2 - Prompt for encryption passphrase at runtime
FR37 → Epic 11 - Apply configuration with priority order
FR38 → Epic 2 - Override security defaults with warning flags
FR39 → Epic 10 - Display Bubbletea crosstown dashboard panel
FR40 → Epic 10 - Log ILP payments to git audit trail
FR41 → Epic 10 - Log structured payment data to JSONL
FR42 → Epic 10 - Export payment history to CSV/JSON
FR43 → Epic 10 - Display payment statistics
FR44 → Epic 10 - Log channel state changes with tx hashes
FR45 → Epic 10 - Display active network prominently
FR46 → Epic 10 - Run Mayor in interactive mode with dashboard
FR47 → Epic 2 + Epic 11 - Manage wallet via CLI commands
FR48 → Epic 4 + Epic 11 - Manage channels via CLI commands
FR49 → Epic 11 - Manage relay connection via CLI
FR50 → Epic 11 - Export payment history via CLI
FR51 → Epic 11 - Multiple output formats (text/JSON/CSV)
FR52 → Epic 11 - Non-interactive mode with --yes flag
FR53 → Epic 11 - Generate shell completion scripts
FR54 → Epic 11 - Automate operations via shell scripts
FR55 → Epic 8 - Create skill files in .claude/skills/
FR56 → Epic 8 - Auto-discover skills and route events
FR57 → Epic 8 - Skills receive raw TOON payloads
FR58 → Epic 8 - Extend Mayor with new event kinds via skills

## Epic List

### Epic 1: Foundation & Package Infrastructure
**Goal:** Development team has the core package structure integrated into gastown, enabling all crosstown functionality.
**FRs covered:** Architecture requirements (internal/ilp/, internal/nostr/, internal/crosstown/ packages)

### Epic 2: Secure Wallet & Key Management
**Goal:** Operators can securely configure, encrypt, and manage wallet credentials for crosstown payments.
**FRs covered:** FR6, FR33, FR35, FR36, FR38, FR47 (wallet commands)
**NFRs:** NFR-S1, NFR-S2, NFR-S5

### Epic 3: Relay Connection & Communication
**Goal:** Mayor can connect to crosstown relay, subscribe to events, and monitor connection health in real-time.
**FRs covered:** FR1, FR2, FR3, FR4, FR5
**NFRs:** NFR-I1, NFR-R3, NFR-R7

### Epic 4: Payment Channel Lifecycle
**Goal:** Operators can open payment channels, monitor balances, validate pricing, and close channels for crosstown payments.
**FRs covered:** FR7, FR8, FR9, FR10, FR11, FR12, FR13, FR14, FR48 (channel CLI commands)
**NFRs:** NFR-S3, NFR-S4, NFR-S7, NFR-I2, NFR-I3, NFR-I9, NFR-R2, NFR-P5

### Epic 5: Job Posting & Payment Flow
**Goal:** Mayor can post DVM jobs to crosstown network via ILP payments and handle payment responses.
**FRs covered:** FR15, FR16, FR17, FR18
**NFRs:** NFR-P1, NFR-R4, NFR-I3

### Epic 6: Job Results & Earnings
**Goal:** Mayor can publish DVM results, receive payments for completed work, and execute jobs in Polecat worktrees.
**FRs covered:** FR24, FR25, FR26
**NFRs:** NFR-P1, NFR-R4

### Epic 7: Job Routing & Correlation
**Goal:** Mayor can route incoming job results to appropriate callback skills using git-backed correlation state.
**FRs covered:** FR19, FR20
**NFRs:** NFR-R1, NFR-SC4

### Epic 8: TOON Event Processing & Skills Integration
**Goal:** Claude Code and skills can process raw TOON events natively without JSON conversion, with automatic skill discovery.
**FRs covered:** FR27, FR28, FR29, FR30, FR31, FR32, FR55, FR56, FR57, FR58
**NFRs:** NFR-P2, NFR-I4, NFR-I8

### Epic 9: Advanced Job Orchestration
**Goal:** Mayor can execute sequential job chains, handle timeouts gracefully, and recover in-flight jobs after crashes.
**FRs covered:** FR21, FR22, FR23
**NFRs:** NFR-R1, NFR-R5, NFR-R6, NFR-R8

### Epic 10: Observability & Audit Trail
**Goal:** Operators can monitor crosstown activity via dashboard, audit all payments in git, and export payment history.
**FRs covered:** FR39, FR40, FR41, FR42, FR43, FR44, FR45, FR46
**NFRs:** NFR-P3, NFR-P4, NFR-P6, NFR-S6, NFR-S8, NFR-I6, NFR-SC5

### Epic 11: CLI Operations & Automation
**Goal:** Operators can manage all crosstown operations via CLI with multiple output formats and automation support.
**FRs covered:** FR34, FR37, FR47 (wallet CLI), FR48 (channel CLI), FR49, FR50, FR51, FR52, FR53, FR54
**NFRs:** NFR-I5, NFR-I10

### Epic 12: Production Readiness & Testing
**Goal:** Development team has comprehensive test coverage (≥50%), E2E Docker tests, and production deployment configuration.
**FRs covered:** Testing infrastructure requirements, build requirements, CI/CD integration
**NFRs:** NFR-SC1, NFR-SC2, NFR-SC3, NFR-SC4, NFR-SC5

## Epic 1: Foundation & Package Infrastructure

Development team has the core package structure integrated into gastown, enabling all crosstown functionality.

### Story 1.1: Set up internal/ilp/ package with BTP/ILP types and stubs

As a **developer**,
I want **the internal/ilp/ package created with core ILP/BTP types, constants, and stub implementations**,
So that **I have the foundation for implementing Interledger payment functionality**.

**Acceptance Criteria:**

**Given** the gastown project exists at the correct Go version (1.24.2)
**When** I navigate to internal/ilp/
**Then** the following files exist with correct structure:
- `types.go` - ILP packet types (PREPARE, FULFILL, REJECT), BTP message types, error types
- `client.go` - BTP WebSocket client stub with interface definition
- `rpc.go` - Base Sepolia RPC client stub with interface definition
- `spsp.go` - SPSP handshake handler stub with interface definition
**And** all files include package documentation comments
**And** all types follow gastown naming conventions (PascalCase for exports)
**And** stub methods return `errors.New("not implemented")` with TODO comments

**Given** the internal/ilp/ package structure exists
**When** I run `go build ./internal/ilp/`
**Then** the package compiles without errors
**And** no external dependencies are imported yet (only stdlib)

### Story 1.2: Set up internal/nostr/ package with NIP-01 types and stubs

As a **developer**,
I want **the internal/nostr/ package created with NIP-01 WebSocket types and TOON encoding stubs**,
So that **I have the foundation for implementing relay communication and event processing**.

**Acceptance Criteria:**

**Given** the gastown project exists
**When** I navigate to internal/nostr/
**Then** the following files exist with correct structure:
- `types.go` - Nostr event types, event kinds (5xxx, 6xxx), filter types, subscription types
- `relay.go` - NIP-01 WebSocket relay client stub with interface definition
- `encoding.go` - TOON encoding/decoding stub functions
**And** event kind constants are defined (KIND_DVM_JOB_REQUEST = 5xxx range, KIND_DVM_RESULT = 6xxx range)
**And** all files include package documentation
**And** stub methods return `errors.New("not implemented")` with TODO comments

**Given** the internal/nostr/ package structure exists
**When** I run `go build ./internal/nostr/`
**Then** the package compiles without errors
**And** only stdlib dependencies are imported (no external deps yet)

### Story 1.3: Set up internal/crosstown/ package with integration layer stubs

As a **developer**,
I want **the internal/crosstown/ package created with channel manager, job tracker, and wallet stubs**,
So that **I have the integration layer that coordinates ILP and Nostr functionality**.

**Acceptance Criteria:**

**Given** the gastown project exists with internal/ilp/ and internal/nostr/ packages
**When** I navigate to internal/crosstown/
**Then** the following files exist with correct structure:
- `types.go` - Crosstown-specific types, constants, error definitions
- `channel_manager.go` - Payment channel state manager stub with interface
- `job_tracker.go` - Job correlation tracker stub with interface
- `wallet.go` - Wallet and key management stub with interface
- `config.go` - Configuration structs matching TOML schema
**And** all files include package documentation
**And** stubs can import internal/ilp/ and internal/nostr/ types
**And** stub methods return `errors.New("not implemented")` with TODO comments

**Given** the internal/crosstown/ package structure exists
**When** I run `go build ./internal/crosstown/`
**Then** the package compiles without errors
**And** imports from internal/ilp/ and internal/nostr/ resolve correctly

### Story 1.4: Add dependencies to go.mod and update Makefile

As a **developer**,
I want **all required external dependencies added to go.mod and build targets updated**,
So that **the project can compile with crosstown packages and their dependencies**.

**Acceptance Criteria:**

**Given** the internal/ilp/, internal/nostr/, and internal/crosstown/ packages exist
**When** I review go.mod
**Then** the following dependencies are added:
- `github.com/gorilla/websocket` (latest stable version)
- `github.com/ALLiDoizCode/toon-go` (latest version)
- `golang.org/x/crypto` (for argon2)
- `github.com/golang/mock` (test dependency only)
**And** `go mod tidy` has been run
**And** `go mod verify` passes

**Given** dependencies are added to go.mod
**When** I review the Makefile
**Then** new make targets exist:
- `make test-crosstown` - runs unit tests for crosstown packages
- `make lint-crosstown` - runs golangci-lint on crosstown packages
**And** existing `make test` target includes crosstown tests
**And** existing `make lint` target includes crosstown linting

**Given** Makefile is updated
**When** I run `make build`
**Then** the build succeeds and includes crosstown packages
**And** no import errors occur

### Story 1.5: Add unit test scaffolding with gomock for all packages

As a **developer**,
I want **test files created for all crosstown packages with gomock setup**,
So that **future stories can immediately write tests for implementations**.

**Acceptance Criteria:**

**Given** internal/ilp/, internal/nostr/, and internal/crosstown/ packages exist
**When** I navigate to each package directory
**Then** test files exist alongside source files:
- `internal/ilp/client_test.go`
- `internal/ilp/rpc_test.go`
- `internal/ilp/spsp_test.go`
- `internal/nostr/relay_test.go`
- `internal/nostr/encoding_test.go`
- `internal/crosstown/channel_manager_test.go`
- `internal/crosstown/job_tracker_test.go`
- `internal/crosstown/wallet_test.go`
**And** each test file contains at least one stub test that passes (e.g., `TestStub` that always passes)

**Given** test files exist
**When** I navigate to internal/ilp/mocks/, internal/nostr/mocks/, internal/crosstown/mocks/
**Then** mock directories exist for generated mocks
**And** a `generate.go` file exists in each package with `//go:generate mockgen` directives
**And** running `go generate ./internal/...` completes without errors

**Given** test scaffolding is complete
**When** I run `make test-crosstown`
**Then** all stub tests pass
**And** test coverage is reported (even if 0% for stubs)
**And** no test compilation errors occur

## Epic 2: Secure Wallet & Key Management

Operators can securely configure, encrypt, and manage wallet credentials for crosstown payments.

### Story 2.1: Implement Argon2 key derivation for wallet encryption

As an **operator**,
I want **secure key derivation using Argon2id for encrypting my wallet private key**,
So that **my encryption keys are derived securely from my passphrase**.

**Acceptance Criteria:**

**Given** the internal/crosstown/wallet.go file exists
**When** I review the key derivation implementation
**Then** the code uses `golang.org/x/crypto/argon2.IDKey()` function
**And** Argon2id parameters are set to: Time=1, Memory=64MB, Threads=4, KeyLen=32 bytes
**And** a cryptographically random salt (32 bytes) is generated per encryption using `crypto/rand.Read()`
**And** the salt is stored alongside the ciphertext for decryption

**Given** the Argon2 key derivation function exists
**When** I call `DeriveKey(passphrase string, salt []byte) ([]byte, error)`
**Then** it returns a 32-byte key suitable for AES-256
**And** the same passphrase + salt always produces the same key
**And** different salts produce different keys even with same passphrase

**Given** unit tests exist in wallet_test.go
**When** I run `go test ./internal/crosstown/`
**Then** TestDeriveKey passes with multiple test cases
**And** test coverage for DeriveKey function is ≥90%

### Story 2.2: Implement wallet encryption and decryption with AES-256-GCM

As an **operator**,
I want **my wallet private key encrypted at rest using AES-256-GCM**,
So that **my private key cannot be stolen if someone accesses my filesystem**.

**Acceptance Criteria:**

**Given** the Argon2 key derivation function exists (Story 2.1)
**When** I review internal/crosstown/wallet.go
**Then** encryption function `EncryptPrivateKey(privateKey string, passphrase string) ([]byte, error)` exists
**And** it uses AES-256-GCM (via `crypto/aes` and `crypto/cipher.NewGCM`)
**And** a random nonce is generated per encryption
**And** the output format is: `[salt(32)][nonce(12)][ciphertext+tag]`

**Given** the encryption function exists
**When** I review the decryption function `DecryptPrivateKey(encrypted []byte, passphrase string) (string, error)`
**Then** it extracts salt, nonce, and ciphertext from the input
**And** it derives the key using the extracted salt and provided passphrase
**And** it decrypts using AES-256-GCM and verifies the authentication tag
**And** it returns an error if authentication fails (tampered data)

**Given** encryption and decryption functions exist
**When** I encrypt a test private key with passphrase "test123"
**Then** the encrypted output differs on each call (different salt/nonce)
**And** decrypting with correct passphrase returns original private key
**And** decrypting with wrong passphrase returns an error
**And** tampering with any byte of ciphertext causes decryption to fail

**Given** unit tests exist in wallet_test.go
**When** I run `go test ./internal/crosstown/`
**Then** TestEncryptDecryptPrivateKey passes
**And** test coverage for encryption/decryption is ≥95%

### Story 2.3: Add wallet configuration via environment variables

As an **operator**,
I want **to configure my wallet using environment variables**,
So that **I can securely provide wallet settings without hardcoding them**.

**Acceptance Criteria:**

**Given** the internal/crosstown/config.go file exists
**When** I review the WalletConfig struct
**Then** it contains fields:
- `Address string` (wallet Ethereum address)
- `EncryptedPrivateKey string` (base64-encoded encrypted key)
- `RPCURL string` (Base Sepolia RPC endpoint)
**And** struct tags include both `toml:` and `env:` mappings

**Given** the WalletConfig struct exists
**When** I set environment variables:
- `GASTOWN_WALLET_ADDRESS=0x1234...`
- `GASTOWN_WALLET_ENCRYPTED_KEY=base64string...`
- `GASTOWN_WALLET_RPC_URL=https://sepolia.base.org`
**Then** the LoadWalletConfig() function reads these values
**And** environment variables take priority over TOML config file
**And** default RPC URL is `https://sepolia.base.org` if not specified

**Given** wallet configuration loading exists
**When** I call `LoadWalletConfig()`
**Then** it validates the wallet address format (0x + 40 hex chars)
**And** it validates EncryptedPrivateKey is valid base64
**And** it validates RPC URL is a valid HTTPS URL
**And** it returns detailed error messages for invalid config

**Given** unit tests exist in config_test.go
**When** I run `go test ./internal/crosstown/`
**Then** TestLoadWalletConfig passes with various env var combinations
**And** test coverage for config loading is ≥85%

### Story 2.4: Implement passphrase prompt at runtime

As an **operator**,
I want **to be prompted for my encryption passphrase when Mayor starts**,
So that **my passphrase is never stored on disk or in environment variables**.

**Acceptance Criteria:**

**Given** the internal/crosstown/wallet.go file exists
**When** I review the passphrase prompt implementation
**Then** a function `PromptForPassphrase() (string, error)` exists
**And** it uses a secure terminal input library (e.g., `golang.org/x/term`)
**And** passphrase input is not echoed to the terminal (hidden input)
**And** the prompt message is: "Enter wallet encryption passphrase: "

**Given** the passphrase prompt function exists
**When** Mayor needs to decrypt the wallet on startup
**Then** it calls PromptForPassphrase() to get the passphrase
**And** the passphrase is stored only in memory during the Mayor session
**And** the passphrase is never logged to files or console output
**And** the passphrase memory is cleared on Mayor shutdown (zeroed out)

**Given** the prompt is displayed
**When** I enter my passphrase and press Enter
**Then** the passphrase is returned to the caller
**And** terminal echo is restored after input
**And** if I press Ctrl+C, the function returns an error and Mayor exits gracefully

**Given** unit tests exist
**When** I run tests with mocked terminal input
**Then** TestPromptForPassphrase passes
**And** passphrase handling does not leak to test output

### Story 2.5: Add CLI command for wallet setup (gt crosstown wallet configure)

As an **operator**,
I want **a CLI command to configure and encrypt my wallet credentials**,
So that **I can securely set up my wallet for crosstown payments**.

**Acceptance Criteria:**

**Given** the internal/cmd/ directory exists
**When** I review internal/cmd/crosstown.go
**Then** a new command group `gt crosstown` exists following Cobra patterns
**And** it integrates with existing gastown command structure
**And** RunE error handling pattern is used (NFR-I5)

**Given** the crosstown command group exists
**When** I run `gt crosstown wallet configure --address 0x1234... --rpc-url https://sepolia.base.org`
**Then** the command prompts: "Enter wallet private key (will be encrypted): "
**And** private key input is hidden (not echoed)
**And** the command prompts: "Enter encryption passphrase: "
**And** passphrase input is hidden
**And** the command prompts: "Confirm passphrase: "
**And** if passphrases don't match, it shows error and re-prompts

**Given** valid wallet credentials are entered
**When** the command completes
**Then** the private key is encrypted using the passphrase
**And** encrypted key is saved to environment variable format in output
**And** the command displays: "Wallet configured successfully. Add to your environment:"
**And** it shows: `export GASTOWN_WALLET_ADDRESS=0x1234...`
**And** it shows: `export GASTOWN_WALLET_ENCRYPTED_KEY=base64string...`
**And** it shows: `export GASTOWN_WALLET_RPC_URL=https://sepolia.base.org`
**And** plaintext private key is never written to disk
**And** plaintext passphrase is never written to disk

**Given** the command is implemented
**When** I run `gt crosstown wallet configure --help`
**Then** usage documentation is displayed
**And** all flags are documented
**And** security warnings are included in help text

### Story 2.6: Add CLI commands for wallet operations (check balance, export key)

As an **operator**,
I want **CLI commands to check my wallet balance and export encrypted keys**,
So that **I can verify my wallet setup and manage credentials**.

**Acceptance Criteria:**

**Given** wallet configuration exists (Stories 2.3-2.5)
**When** I run `gt crosstown wallet balance`
**Then** the command reads wallet address from environment variables
**And** it connects to Base Sepolia RPC endpoint
**And** it queries the M2M token balance for the wallet address
**And** it displays: "Wallet: 0x1234... Balance: 50.00 M2M tokens (Base Sepolia)"
**And** if wallet not configured, it shows helpful error with setup instructions

**Given** wallet is configured
**When** I run `gt crosstown wallet export`
**Then** it prompts for encryption passphrase
**And** it decrypts the private key
**And** it displays: "⚠️  WARNING: Private key will be shown. Ensure screen is not shared."
**And** it prompts: "Type YES to continue: "
**And** if I type "YES", it displays the decrypted private key
**And** if I type anything else, it aborts with "Export cancelled"
**And** the private key display includes warning to save securely

**Given** wallet commands support output formats (NFR-I5)
**When** I run `gt crosstown wallet balance --format json`
**Then** output is valid JSON: `{"address": "0x1234...", "balance": "50.00", "network": "base-sepolia"}`
**And** `--format csv` outputs CSV format
**And** default format is human-readable text

**Given** unit tests exist for CLI commands
**When** I run tests
**Then** command parsing and validation tests pass
**And** RPC client is mocked for balance queries

### Story 2.7: Enforce encryption at rest and security validation

As an **operator**,
I want **Mayor to refuse to start if my wallet private key is unencrypted**,
So that **I am protected from accidentally using insecure wallet storage**.

**Acceptance Criteria:**

**Given** Mayor startup sequence exists
**When** Mayor initializes wallet configuration
**Then** it checks if `GASTOWN_WALLET_ENCRYPTED_KEY` is set
**And** it attempts to decode from base64
**And** it validates the encrypted key format: `[salt][nonce][ciphertext+tag]`
**And** if key format is invalid or missing, Mayor refuses to start

**Given** an unencrypted private key is detected (e.g., raw hex string)
**When** Mayor startup attempts to load wallet
**Then** it displays error: "🚨 SECURITY ERROR: Wallet private key appears unencrypted"
**And** it displays: "Run 'gt crosstown wallet configure' to encrypt your key"
**And** it displays: "Or set GASTOWN_ALLOW_UNENCRYPTED_KEYS=true to override (NOT RECOMMENDED)"
**And** Mayor exits with error code 1

**Given** security validation fails but operator wants to override (FR38)
**When** I set `GASTOWN_ALLOW_UNENCRYPTED_KEYS=true` and start Mayor
**Then** Mayor displays: "⚠️  WARNING: Running with unencrypted wallet key (insecure!)"
**And** the warning is logged to Mayor's output
**And** Mayor proceeds to start
**And** the warning persists in dashboard display

**Given** encrypted wallet files are created
**When** Mayor creates wallet state files
**Then** all files have Unix permissions 0600 (owner read/write only)
**And** directories have permissions 0700 (owner read/write/execute only)
**And** permission setting is verified after file creation
**And** if permission setting fails, Mayor logs an error

**Given** integration tests exist
**When** I run wallet security validation tests
**Then** tests verify encryption enforcement
**And** tests verify override flag behavior
**And** tests verify file permissions are correctly set

## Epic 3: Relay Connection & Communication

Mayor can connect to crosstown relay, subscribe to events, and monitor connection health in real-time.

### Story 3.1: Implement NIP-01 WebSocket client with gorilla/websocket

As a **Mayor agent**,
I want **to connect to crosstown relay using NIP-01 WebSocket protocol**,
So that **I can send and receive Nostr events for crosstown job distribution**.

**Acceptance Criteria:**

**Given** the internal/nostr/relay.go file exists with stub implementation
**When** I review the RelayClient implementation
**Then** it uses `github.com/gorilla/websocket` for WebSocket connections
**And** it implements the RelayClient interface with methods:
- `Connect(url string) error`
- `SendMessage(msg []byte) error`
- `ReceiveMessage() ([]byte, error)`
- `Close() error`
**And** connection uses proper WebSocket headers and handshake

**Given** the RelayClient is implemented
**When** I call `Connect("ws://relay.crosstown.network")`
**Then** the client establishes a WebSocket connection to the relay
**And** the connection state is tracked (disconnected → connecting → connected)
**And** connection errors are returned with descriptive messages
**And** the client sets appropriate timeouts (connect: 10s, read: 60s, write: 10s)

**Given** a successful WebSocket connection exists
**When** I send a NIP-01 REQ message via SendMessage()
**Then** the message is sent as a JSON array following NIP-01 format: `["REQ", subscription_id, filter]`
**And** the write operation respects the write timeout
**And** write errors are captured and returned

**Given** the relay sends events
**When** I call ReceiveMessage()
**Then** it reads raw JSON messages from the WebSocket
**And** it respects the read timeout
**And** read errors (timeout, connection closed) are captured and returned
**And** the raw message bytes are returned without parsing (zero-copy)

**Given** unit tests exist in relay_test.go
**When** I run `go test ./internal/nostr/`
**Then** TestRelayClientConnect passes with mocked WebSocket server
**And** TestSendReceiveMessage passes
**And** test coverage for relay.go is ≥85%

### Story 3.2: Implement subscription filters for Nostr event kinds

As a **Mayor agent**,
I want **to subscribe to specific Nostr event kinds with filters**,
So that **I receive only DVM job requests (kind:5xxx) and results (kind:6xxx) relevant to me**.

**Acceptance Criteria:**

**Given** the internal/nostr/types.go file exists
**When** I review the Filter struct
**Then** it contains NIP-01 filter fields:
- `Kinds []int` (event kinds to match)
- `Authors []string` (optional author pubkeys)
- `IDs []string` (optional event IDs)
- `Since *int64` (optional timestamp)
- `Until *int64` (optional timestamp)
**And** the struct has a `ToJSON() ([]byte, error)` method for serialization

**Given** the RelayClient exists (Story 3.1)
**When** I review the Subscribe method signature
**Then** it is defined as: `Subscribe(id string, filter Filter) error`
**And** it sends a NIP-01 REQ message with format: `["REQ", id, filter]`
**And** the filter is serialized to JSON matching NIP-01 specification

**Given** I want to subscribe to DVM job requests
**When** I call `Subscribe("jobs", Filter{Kinds: []int{5000, 5001, 5002}})`
**Then** the relay client sends: `["REQ", "jobs", {"kinds": [5000, 5001, 5002]}]`
**And** the subscription ID "jobs" is stored in active subscriptions map
**And** the filter is associated with the subscription for tracking

**Given** I want to subscribe to DVM results by job ID
**When** I call `Subscribe("results", Filter{Kinds: []int{6000}, IDs: ["job-abc123"]})`
**Then** the relay client sends: `["REQ", "results", {"kinds": [6000], "ids": ["job-abc123"]}]`
**And** the subscription is tracked separately from other subscriptions

**Given** the relay responds with EOSE (end of stored events)
**When** the client receives: `["EOSE", "jobs"]`
**Then** it parses the EOSE message
**And** it marks the subscription "jobs" as synchronized (stored events complete)
**And** it logs: "Subscription 'jobs' synchronized with relay"

**Given** unit tests exist in relay_test.go
**When** I run `go test ./internal/nostr/`
**Then** TestSubscribe passes with various filter combinations
**And** TestFilterSerialization verifies JSON output matches NIP-01 spec
**And** test coverage for subscription logic is ≥90%

### Story 3.3: Implement connection status tracking and state management

As a **Mayor agent**,
I want **to track relay connection status with state transitions**,
So that **I can monitor connection health and react to state changes**.

**Acceptance Criteria:**

**Given** the internal/nostr/relay.go file exists
**When** I review the ConnectionState type
**Then** it is defined as an enum/constants:
- `StateDisconnected` (initial state)
- `StateConnecting` (connection in progress)
- `StateConnected` (active connection)
- `StateReconnecting` (reconnection in progress)
- `StateError` (connection failed)
**And** the RelayClient has a `GetState() ConnectionState` method

**Given** the RelayClient tracks connection state
**When** I call `Connect()` and the connection succeeds
**Then** state transitions: Disconnected → Connecting → Connected
**And** each state transition is logged with timestamp
**And** `GetState()` returns `StateConnected`

**Given** the connection fails during Connect()
**When** the WebSocket handshake fails (e.g., network error)
**Then** state transitions: Disconnected → Connecting → Error
**And** the error details are stored and retrievable via `GetLastError() error`
**And** `GetState()` returns `StateError`

**Given** a connected relay client
**When** the connection is lost (relay closes connection)
**Then** state transitions: Connected → Disconnected
**And** a connection lost event is logged with reason
**And** active subscriptions are marked as inactive

**Given** the relay client provides connection status info
**When** I call `GetConnectionInfo()`
**Then** it returns a struct with:
- `State ConnectionState`
- `URL string` (relay URL)
- `ConnectedAt time.Time` (connection timestamp)
- `LastError error` (most recent error)
- `ActiveSubscriptions []string` (list of subscription IDs)
**And** all fields are populated accurately

**Given** unit tests exist in relay_test.go
**When** I run `go test ./internal/nostr/`
**Then** TestConnectionStateTransitions passes
**And** TestGetConnectionInfo verifies all fields
**And** test coverage for state management is ≥90%

### Story 3.4: Implement exponential backoff reconnection logic

As a **Mayor agent**,
I want **automatic reconnection with exponential backoff when the relay connection is lost**,
So that **transient network issues don't permanently disconnect me from the crosstown network**.

**Acceptance Criteria:**

**Given** the internal/nostr/relay.go file exists
**When** I review the reconnection configuration
**Then** exponential backoff parameters are defined:
- Initial delay: 1 second
- Maximum delay: 30 seconds
- Backoff multiplier: 2.0
- Jitter: random ±20% to prevent thundering herd (NFR-R3)
**And** parameters are configurable via RelayClientConfig

**Given** the RelayClient is connected
**When** the connection is lost (relay closes, network error)
**Then** the client automatically attempts reconnection
**And** state transitions to `StateReconnecting`
**And** first reconnection attempt happens after 1 second + jitter

**Given** the first reconnection attempt fails
**When** the client retries
**Then** second attempt happens after 2 seconds + jitter
**And** third attempt happens after 4 seconds + jitter
**And** fourth attempt happens after 8 seconds + jitter
**And** subsequent attempts happen after 30 seconds + jitter (capped at max)

**Given** reconnection succeeds on attempt N
**When** the connection is re-established
**Then** state transitions to `StateConnected`
**And** all previous subscriptions are re-sent to the relay automatically
**And** reconnection delay resets to initial value (1 second)
**And** reconnection success is logged with attempt count

**Given** jitter is applied to prevent thundering herd
**When** calculating reconnection delay
**Then** actual delay is: `baseDelay * (1.0 + random(-0.2, 0.2))`
**And** jitter ensures multiple clients don't reconnect simultaneously

**Given** the operator wants to disable auto-reconnect
**When** RelayClient is configured with `AutoReconnect: false`
**Then** connection loss does NOT trigger automatic reconnection
**And** state transitions to `StateDisconnected`
**And** the client must be manually reconnected via Connect()

**Given** unit tests exist in relay_test.go
**When** I run `go test ./internal/nostr/`
**Then** TestExponentialBackoff verifies delay calculations
**And** TestReconnectionFlow simulates connection loss and recovery
**And** TestResubscribeOnReconnect verifies subscriptions are restored
**And** test coverage for reconnection logic is ≥90%

### Story 3.5: Handle graceful degradation when relay is unavailable

As a **Mayor agent**,
I want **to continue operating in local-only mode when the relay is unavailable**,
So that **Polecat orchestration continues even without crosstown network access** (NFR-R7).

**Acceptance Criteria:**

**Given** the Mayor startup sequence exists
**When** Mayor attempts to connect to crosstown relay on startup
**Then** if connection fails, Mayor does NOT exit
**And** Mayor logs: "⚠️  Crosstown relay unavailable - operating in local-only mode"
**And** Mayor continues with local Polecat orchestration
**And** relay connection continues attempting reconnection in background

**Given** Mayor is running in local-only mode
**When** I check Mayor's operational status
**Then** local Polecat spawning and orchestration work normally
**And** git worktree operations work normally
**And** only crosstown-specific features are disabled (job posting, network distribution)

**Given** Mayor is in local-only mode (relay disconnected)
**When** the relay becomes available later
**Then** auto-reconnection logic (Story 3.4) connects successfully
**And** Mayor transitions from local-only to crosstown-enabled mode
**And** Mayor logs: "✅ Crosstown relay connected - network job distribution enabled"
**And** subscriptions are established automatically

**Given** the operator checks connection status during local-only mode
**When** they run `gt crosstown relay status`
**Then** the command displays:
- "Status: Disconnected (local-only mode)"
- "Relay: ws://relay.crosstown.network"
- "Reconnection: Active (next attempt in Xs)"
- "Local operations: Available"
**And** the status clearly indicates local mode is operational

**Given** integration tests exist
**When** I run tests with relay unavailable
**Then** TestGracefulDegradation verifies Mayor starts successfully
**And** TestLocalOnlyMode verifies Polecat operations work
**And** test coverage for degradation handling is ≥85%

### Story 3.6: Add CLI commands for relay connection management

As an **operator**,
I want **CLI commands to manage relay connections**,
So that **I can configure, connect, disconnect, and monitor relay connectivity** (FR49).

**Acceptance Criteria:**

**Given** the internal/cmd/crosstown.go command group exists (Story 2.5)
**When** I review the relay subcommands
**Then** the following commands exist:
- `gt crosstown relay connect --url <relay-url>`
- `gt crosstown relay disconnect`
- `gt crosstown relay status`
- `gt crosstown relay subscribe --kinds <kinds>`

**Given** the relay connect command exists
**When** I run `gt crosstown relay connect --url ws://relay.crosstown.network`
**Then** Mayor attempts to connect to the specified relay URL
**And** connection progress is displayed: "Connecting to ws://relay.crosstown.network..."
**And** on success: "✅ Connected to crosstown relay"
**And** on failure: "❌ Connection failed: <error details>"
**And** the relay URL is saved to config for future use

**Given** the relay disconnect command exists
**When** I run `gt crosstown relay disconnect`
**Then** Mayor closes the active relay connection
**And** auto-reconnection is disabled
**And** the command displays: "Relay connection closed. Mayor in local-only mode."
**And** active subscriptions are cleared

**Given** the relay status command exists
**When** I run `gt crosstown relay status`
**Then** it displays connection information:
- "Status: Connected" or "Disconnected" or "Reconnecting"
- "Relay URL: ws://relay.crosstown.network"
- "Connected since: 2026-02-18 14:23:45"
- "Active subscriptions: jobs, results (2 total)"
- "Messages sent: 42, received: 128"
**And** status includes reconnection info if applicable

**Given** the relay subscribe command exists
**When** I run `gt crosstown relay subscribe --kinds 5000,6000`
**Then** Mayor subscribes to event kinds 5000 and 6000
**And** a subscription ID is auto-generated
**And** the command displays: "✅ Subscribed to kinds: 5000, 6000 (subscription: <id>)"
**And** received events matching the filter are logged

**Given** all relay commands support output formats
**When** I run `gt crosstown relay status --format json`
**Then** output is valid JSON with all status fields
**And** `--format csv` outputs CSV format

**Given** unit tests exist for CLI commands
**When** I run tests
**Then** command parsing and validation tests pass
**And** relay client is mocked for command execution

## Epic 4: Payment Channel Lifecycle

Operators can open payment channels, monitor balances, validate pricing, and close channels for crosstown payments.

### Story 4.1: Implement SPSP handshake with crosstown connector

As a **Mayor agent**,
I want **to perform SPSP handshake with the crosstown connector**,
So that **I can discover payment endpoint details and negotiate settlement parameters** (FR7, FR8).

**Acceptance Criteria:**

**Given** the internal/ilp/spsp.go file exists with stub implementation
**When** I review the SPSP client implementation
**Then** it implements `PerformHandshake(connectorURL string) (*SPSPResponse, error)`
**And** it uses net/http to perform HTTPS GET request to connector's SPSP endpoint
**And** it follows SPSP specification for endpoint discovery

**Given** the SPSP client makes a handshake request
**When** I call `PerformHandshake("https://connector.crosstown.network/.well-known/pay")`
**Then** it sends HTTP GET with Accept header: "application/spsp4+json"
**And** it includes User-Agent: "gastown/v0.7.0 (crosstown)"
**And** request timeout is set to 10 seconds

**Given** the connector responds with SPSP information
**When** the response is received
**Then** it parses JSON response with fields:
- `destination_account` (ILP address of connector)
- `shared_secret` (base64 encoded shared secret for BTP auth)
- `receipts_enabled` (boolean)
- `balance_minimum` (minimum balance in base units)
- `balance_maximum` (maximum balance in base units)
- `asset_code` (M2M)
- `asset_scale` (token decimals)
**And** all required fields are validated for presence

**Given** SPSP response includes settlement chain info
**When** I review the settlement chain negotiation
**Then** the response includes custom field: `settlement_chain` (e.g., "evm:base:84532")
**And** Mayor validates the settlement chain matches expected: "evm:base:84532" (Base Sepolia)
**And** if settlement chain doesn't match, Mayor returns error with helpful message
**And** if settlement chain matches, negotiation succeeds

**Given** SPSP response includes pricing information
**When** I extract the basePricePerByte value
**Then** it is validated against threshold (default: ≤100 units/byte per NFR-S7)
**And** if price exceeds threshold, Mayor logs warning: "⚠️  High pricing detected: XYZ units/byte"
**And** operator is notified but handshake continues (warning, not error)

**Given** unit tests exist in spsp_test.go
**When** I run `go test ./internal/ilp/`
**Then** TestSPSPHandshake passes with mocked HTTP server
**And** TestSettlementChainValidation verifies chain matching
**And** TestPriceValidation verifies threshold checking
**And** test coverage for spsp.go is ≥90%

### Story 4.2: Implement BTP WebSocket client for connector communication

As a **Mayor agent**,
I want **a BTP WebSocket client to communicate with the crosstown connector**,
So that **I can send and receive ILP packets over BTP transport** (NFR-I2).

**Acceptance Criteria:**

**Given** the internal/ilp/client.go file exists with stub implementation
**When** I review the BTPClient implementation
**Then** it uses `github.com/gorilla/websocket` for WebSocket connections
**And** it implements the BTPClient interface with methods:
- `Connect(url string, authToken string) error`
- `SendPacket(packet []byte) error`
- `ReceivePacket() ([]byte, error)`
- `Close() error`
**And** it follows BTP specification for packet framing (NFR-I2)

**Given** SPSP handshake provided shared_secret (Story 4.1)
**When** I call `Connect("wss://connector.crosstown.network:3000", sharedSecret)`
**Then** the client establishes WebSocket connection with wss:// (TLS required per NFR-S3)
**And** it sends BTP auth message with shared secret as bearer token
**And** it validates TLS certificate of connector
**And** connection timeout is 10 seconds

**Given** BTP connection is established
**When** I send an ILP packet via `SendPacket(ilpData)`
**Then** it wraps the packet in BTP framing: `[type(1)][requestId(4)][data]`
**And** type is BTP_MESSAGE (0x06) for ILP packets
**And** requestId is auto-incremented for each packet
**And** the framed packet is sent over WebSocket

**Given** the connector sends a BTP response
**When** I call `ReceivePacket()`
**Then** it reads the BTP frame from WebSocket
**And** it extracts type, requestId, and ILP packet data
**And** it validates the frame structure matches BTP spec
**And** it returns the raw ILP packet bytes (unwrapped from BTP frame)

**Given** TLS certificate validation is enforced (NFR-S3)
**When** connecting to a connector with invalid certificate
**Then** the connection fails with certificate validation error
**And** Mayor logs: "🚨 TLS certificate validation failed for connector"
**And** the operator is instructed to verify connector certificate

**Given** unit tests exist in client_test.go
**When** I run `go test ./internal/ilp/`
**Then** TestBTPConnect passes with mocked WebSocket server
**And** TestBTPFraming verifies packet framing/unframing
**And** TestTLSValidation verifies certificate checking
**And** test coverage for client.go is ≥85%

### Story 4.3: Implement Base Sepolia RPC client for on-chain operations

As a **Mayor agent**,
I want **an RPC client to interact with Base Sepolia blockchain**,
So that **I can open payment channels, query balances, and read on-chain state** (NFR-I9).

**Acceptance Criteria:**

**Given** the internal/ilp/rpc.go file exists with stub implementation
**When** I review the RPCClient implementation
**Then** it uses `net/http` for JSON-RPC calls to Base Sepolia
**And** it implements methods:
- `GetBalance(address string) (*big.Int, error)`
- `OpenChannel(toAddress string, amount *big.Int) (txHash string, error)`
- `CloseChannel(channelID string) (txHash string, error)`
- `GetChannelState(channelID string) (*ChannelState, error)`
**And** default RPC endpoint is "https://sepolia.base.org"

**Given** the RPC client queries token balance
**When** I call `GetBalance("0x1234...")`
**Then** it sends JSON-RPC request: `eth_call` with M2M token contract
**And** it calls balanceOf(address) method on M2M token contract
**And** it parses the response as uint256 balance
**And** it returns the balance as *big.Int

**Given** the RPC client opens a payment channel
**When** I call `OpenChannel("0xConnectorAddress", big.NewInt(50e18))`
**Then** it constructs an Ethereum transaction:
- To: M2M payment channel contract address
- Function: openChannel(address connector, uint256 amount)
- Amount: 50 M2M tokens (converted to wei)
**And** it signs the transaction with wallet private key (from Story 2.2)
**And** it sends via `eth_sendRawTransaction`
**And** it returns the transaction hash

**Given** balance limits are enforced (NFR-S4)
**When** I attempt to open a channel with >100 M2M tokens (testnet limit)
**Then** the OpenChannel call returns error before sending transaction
**And** error message: "Channel deposit exceeds maximum limit (100 M2M for testnet)"
**And** the configurable limit is read from GASTOWN_MAX_CHANNEL_BALANCE_M2M env var

**Given** RPC operations include retry logic
**When** an RPC call fails with transient network error
**Then** the client retries with exponential backoff (1s, 2s, 4s)
**And** maximum 3 retry attempts before failing
**And** permanent errors (invalid params) are not retried

**Given** unit tests exist in rpc_test.go
**When** I run `go test ./internal/ilp/`
**Then** TestGetBalance passes with mocked RPC responses
**And** TestOpenChannel verifies transaction construction
**And** TestBalanceLimits verifies NFR-S4 enforcement
**And** test coverage for rpc.go is ≥85%

### Story 4.4: Implement payment channel opening with M2M token deposit

As an **operator**,
I want **to open a payment channel with an M2M token deposit**,
So that **I can fund crosstown payments to the relay** (FR9, NFR-P5).

**Acceptance Criteria:**

**Given** SPSP handshake completed (Story 4.1) and BTP client connected (Story 4.2) and RPC client exists (Story 4.3)
**When** I initiate channel opening via `OpenPaymentChannel(connectorURL string, depositAmount *big.Int)`
**Then** the process executes in sequence:
1. Perform SPSP handshake to get connector address and settlement chain
2. Validate settlement chain is Base Sepolia (84532)
3. Validate deposit amount against limits (NFR-S4)
4. Call RPC OpenChannel to send on-chain transaction
5. Wait for transaction confirmation (1 block)
6. Extract channel ID from transaction logs
7. Return channel ID and initial state

**Given** channel opening is initiated
**When** the on-chain transaction is broadcast
**Then** it completes within 30 seconds (NFR-P5)
**And** if it takes >30 seconds, Mayor logs warning but continues waiting
**And** operator sees progress: "Opening channel... waiting for confirmation..."

**Given** the transaction is confirmed on-chain
**When** the channel opening succeeds
**Then** a unique channel ID is generated from transaction hash
**And** initial channel state is created:
- `ChannelID`: derived from tx hash
- `Status`: "open"
- `Balance`: depositAmount (in M2M tokens)
- `SettlementChain`: "evm:base:84532"
- `SettlementAddress`: connector address
- `OpenedAt`: current timestamp
- `ConnectorBTP`: BTP WebSocket URL
- `RelayURL`: relay WebSocket URL

**Given** channel opening succeeds
**When** the function returns
**Then** it returns `(*Channel, error)` with populated Channel struct
**And** success is logged: "✅ Payment channel opened: ch_abc123 (50.00 M2M)"

**Given** channel opening fails (insufficient balance, network error)
**When** the transaction fails
**Then** detailed error is returned with failure reason
**And** no channel state is created
**And** operator sees error: "❌ Channel opening failed: <reason>"

**Given** unit tests exist in channel_manager_test.go
**When** I run `go test ./internal/crosstown/`
**Then** TestOpenPaymentChannel passes with mocked dependencies
**And** TestChannelOpeningTimeout verifies 30s requirement
**And** test coverage for channel opening is ≥85%

### Story 4.5: Implement channel state persistence in git worktree (TOML files)

As a **Mayor agent**,
I want **payment channel state persisted in git worktree as TOML files**,
So that **channel state survives Mayor crashes and provides audit trail** (FR10, NFR-S6).

**Acceptance Criteria:**

**Given** a payment channel is opened (Story 4.4)
**When** the channel state needs to be persisted
**Then** it is saved to: `~/.gt/hooks/mayor-{id}/crosstown/channels/ch_{channel_id}.toml`
**And** the directory structure is created if it doesn't exist
**And** file permissions are set to 0600 (owner read/write only per NFR-S5)
**And** directory permissions are set to 0700 (owner access only)

**Given** channel state is written to TOML file
**When** I review the file format
**Then** it contains all channel fields:
```toml
channel_id = "ch_7f3a9b2e"
status = "open"
balance_m2m = "50.00"
settlement_chain = "evm:base:84532"
settlement_address = "0x39eaF99Cd4965A28DFe8B1455DD42aB49D0836B9"
opened_at = "2026-02-18T14:00:00Z"
connector_btp = "wss://connector.crosstown.network:3000"
relay_url = "ws://relay.crosstown.network"
```
**And** all values are properly escaped and formatted as valid TOML

**Given** channel state file is created
**When** the state is committed to git
**Then** a git commit is created with message: "Payment channel opened: ch_7f3a9b2e (50.00 M2M)"
**And** the commit includes the channel TOML file
**And** git commit provides tamper-evident audit trail (NFR-S6)

**Given** channel state is updated (balance change, status change)
**When** the state file is modified
**Then** the TOML file is updated with new values
**And** a new git commit is created with descriptive message
**And** example: "Payment channel ch_7f3a9b2e: balance 50.00 → 49.87 M2M (job-5abc123)"
**And** git history provides complete audit trail of all state changes

**Given** Mayor needs to load channel state
**When** `LoadChannelState(channelID string)` is called
**Then** it reads the TOML file from git worktree
**And** it parses TOML into Channel struct
**And** it validates all required fields are present
**And** it returns error if file doesn't exist or parsing fails

**Given** git worktree locking exists (NFR-R6)
**When** channel state operations execute
**Then** all git operations are serialized (not concurrent)
**And** gastown's existing git worktree locking mechanism is used
**And** no race conditions occur with concurrent channel updates

**Given** unit tests exist in channel_manager_test.go
**When** I run `go test ./internal/crosstown/`
**Then** TestPersistChannelState passes
**And** TestLoadChannelState verifies TOML parsing
**And** TestFilePermissions verifies 0600 enforcement
**And** test coverage for persistence is ≥90%

### Story 4.6: Implement channel balance monitoring with threshold warnings

As an **operator**,
I want **automatic monitoring of payment channel balance with configurable warnings**,
So that **I'm alerted when balance drops below threshold and can top up before running out** (FR11).

**Acceptance Criteria:**

**Given** the internal/crosstown/channel_manager.go file exists
**When** I review the balance monitoring implementation
**Then** a `MonitorChannelBalance(channelID string)` function exists
**And** it periodically checks channel balance (every 60 seconds)
**And** it compares balance against warning threshold (default: 10 M2M tokens)
**And** threshold is configurable via GASTOWN_CHANNEL_BALANCE_WARNING_M2M env var

**Given** channel balance monitoring is active
**When** the balance drops below threshold (e.g., 9.5 M2M remaining)
**Then** Mayor logs warning: "⚠️  Channel ch_abc123 balance low: 9.50 M2M (threshold: 10.00 M2M)"
**And** warning is displayed in dashboard (if visible)
**And** warning is logged to payment log file
**And** subsequent warnings are rate-limited (max 1 per 5 minutes)

**Given** balance is critically low (< 1 M2M)
**When** monitoring detects critical balance
**Then** Mayor logs critical alert: "🚨 CRITICAL: Channel ch_abc123 nearly depleted: 0.80 M2M"
**And** alert is displayed prominently in dashboard
**And** operator is advised: "Top up channel or payments will fail"

**Given** balance monitoring checks current balance
**When** `GetCurrentBalance(channelID string)` is called
**Then** it reads the latest balance from channel state TOML file
**And** optionally queries on-chain balance for verification
**And** it returns balance as decimal (e.g., 49.87 M2M)

**Given** balance has been updated after a payment
**When** the channel state is persisted (Story 4.5)
**Then** the monitoring system picks up the new balance
**And** threshold check is performed immediately
**And** warnings are triggered if needed

**Given** unit tests exist in channel_manager_test.go
**When** I run `go test ./internal/crosstown/`
**Then** TestMonitorChannelBalance passes
**And** TestBalanceWarnings verifies threshold logic
**And** TestBalanceRateLimiting verifies warning deduplication
**And** test coverage for monitoring is ≥85%

### Story 4.7: Implement payment channel closing and withdrawal

As an **operator**,
I want **to close payment channels and withdraw remaining funds**,
So that **I can retrieve my M2M tokens when done using crosstown** (FR12).

**Acceptance Criteria:**

**Given** an open payment channel exists (Story 4.4)
**When** I call `ClosePaymentChannel(channelID string)`
**Then** the process executes in sequence:
1. Load current channel state from git worktree
2. Verify channel status is "open"
3. Call RPC CloseChannel to send on-chain close transaction
4. Wait for transaction confirmation (1 block)
5. Update channel state: status = "closed", closed_at = timestamp
6. Persist updated state to git with commit message
7. Return final balance and transaction hash

**Given** channel closing transaction is sent
**When** the on-chain transaction is broadcast
**Then** it calls closeChannel(channelID) on the payment channel contract
**And** remaining balance is transferred back to operator's wallet
**And** operator sees progress: "Closing channel... waiting for confirmation..."

**Given** the close transaction is confirmed
**When** channel closing succeeds
**Then** channel state file is updated:
```toml
status = "closed"
closed_at = "2026-02-18T16:30:00Z"
final_balance_m2m = "35.42"
close_tx_hash = "0xabc..."
```
**And** git commit is created: "Payment channel closed: ch_abc123 (withdrew 35.42 M2M)"
**And** success is logged: "✅ Channel ch_abc123 closed. Withdrew 35.42 M2M tokens."

**Given** operator wants to close all channels
**When** I call `CloseAllChannels()`
**Then** it iterates through all open channels in crosstown/channels/ directory
**And** it closes each channel sequentially
**And** it reports total withdrawn: "Closed 3 channels. Total withdrawn: 105.67 M2M"

**Given** channel closing fails (network error, contract error)
**When** the transaction fails
**Then** detailed error is returned with failure reason
**And** channel state remains "open"
**And** operator can retry closing

**Given** unit tests exist in channel_manager_test.go
**When** I run `go test ./internal/crosstown/`
**Then** TestClosePaymentChannel passes with mocked RPC
**And** TestCloseAllChannels verifies batch closing
**And** test coverage for closing is ≥85%

### Story 4.8: Implement on-chain state reconstruction from blockchain events

As a **Mayor agent**,
I want **to reconstruct payment channel state from blockchain if git state is lost**,
So that **I can recover from git repository corruption or accidental deletion** (FR13, NFR-R2).

**Acceptance Criteria:**

**Given** the internal/ilp/rpc.go has channel query methods
**When** I review the on-chain state reconstruction
**Then** a function `ReconstructChannelFromChain(channelID string) (*Channel, error)` exists
**And** it queries Base Sepolia blockchain for channel events
**And** it retrieves channel open/close events from M2M payment channel contract

**Given** I lost git worktree channel state files
**When** I call `ReconstructChannelFromChain("ch_abc123")`
**Then** it queries blockchain logs for:
- `ChannelOpened` event with matching channelID
- `ChannelClosed` event (if channel was closed)
- `PaymentMade` events to calculate current balance
**And** it reconstructs the Channel struct from on-chain data

**Given** on-chain events are retrieved
**When** reconstructing channel state
**Then** it extracts from events:
- `ChannelID`: from event log
- `Balance`: initial deposit minus sum of PaymentMade events
- `SettlementAddress`: from ChannelOpened event
- `OpenedAt`: timestamp from ChannelOpened block
- `ClosedAt`: timestamp from ChannelClosed block (if closed)
- `Status`: "open" or "closed" based on events

**Given** channel state is reconstructed from chain
**When** reconstruction completes successfully
**Then** it creates a new TOML file in git worktree with reconstructed state
**And** it commits to git with message: "Reconstructed channel state from on-chain: ch_abc123"
**And** success is logged: "✅ Channel state reconstructed from blockchain"

**Given** operator wants to reconstruct all lost channels
**When** I call `ReconstructAllChannels(walletAddress string)`
**Then** it queries all ChannelOpened events where sender = walletAddress
**And** it reconstructs state for each discovered channel
**And** it reports: "Reconstructed 2 channels from blockchain"

**Given** blockchain is unavailable or query fails
**When** reconstruction is attempted
**Then** detailed error is returned
**And** operator is advised to retry when blockchain is accessible

**Given** unit tests exist in rpc_test.go
**When** I run `go test ./internal/ilp/`
**Then** TestReconstructChannelFromChain passes with mocked events
**And** TestReconstructAllChannels verifies batch reconstruction
**And** test coverage for reconstruction is ≥85%

### Story 4.9: Add CLI commands for channel management

As an **operator**,
I want **CLI commands to manage payment channels**,
So that **I can open, monitor, close, and list channels via command line** (FR48).

**Acceptance Criteria:**

**Given** the internal/cmd/crosstown.go command group exists
**When** I review the channel subcommands
**Then** the following commands exist:
- `gt crosstown channel open --connector <url> --amount <M2M>`
- `gt crosstown channel status <channel-id>`
- `gt crosstown channel close <channel-id>`
- `gt crosstown channel list`
- `gt crosstown channel reconstruct [--channel-id <id> | --all]`

**Given** the channel open command exists
**When** I run `gt crosstown channel open --connector https://connector.crosstown.network --amount 50`
**Then** it performs SPSP handshake with connector
**And** it opens payment channel with 50 M2M tokens
**And** it displays progress: "Opening channel with 50.00 M2M..."
**And** on success: "✅ Channel opened: ch_abc123 (50.00 M2M on Base Sepolia)"
**And** on failure: "❌ Channel opening failed: <error>"

**Given** the channel status command exists
**When** I run `gt crosstown channel status ch_abc123`
**Then** it displays channel information:
```
Channel: ch_abc123
Status: Open
Balance: 35.42 M2M tokens
Settlement: Base Sepolia (84532)
Connector: wss://connector.crosstown.network:3000
Opened: 2026-02-18 14:00:00
```
**And** if channel not found: "❌ Channel not found: ch_abc123"

**Given** the channel close command exists
**When** I run `gt crosstown channel close ch_abc123`
**Then** it prompts for confirmation: "Close channel ch_abc123 and withdraw 35.42 M2M? (yes/no): "
**And** if confirmed, it closes the channel
**And** displays: "✅ Channel closed. Withdrew 35.42 M2M tokens. Tx: 0xabc..."

**Given** the channel list command exists
**When** I run `gt crosstown channel list`
**Then** it displays all channels:
```
CHANNEL ID    STATUS  BALANCE      SETTLEMENT       OPENED
ch_abc123     Open    35.42 M2M    Base Sepolia     2026-02-18 14:00
ch_def456     Closed  0.00 M2M     Base Sepolia     2026-02-17 10:30
```
**And** if no channels exist: "No payment channels found"

**Given** the channel reconstruct command exists
**When** I run `gt crosstown channel reconstruct --all`
**Then** it reconstructs all channels from blockchain
**And** displays: "Reconstructed 2 channels from on-chain events"

**Given** all channel commands support output formats
**When** I run `gt crosstown channel list --format json`
**Then** output is valid JSON array of channel objects
**And** `--format csv` outputs CSV format

**Given** unit tests exist for CLI commands
**When** I run tests
**Then** command parsing and execution tests pass
**And** channel manager is mocked for command execution

## Epic 5: Job Posting & Payment Flow

Mayor can post DVM jobs to crosstown network via ILP payments and handle payment responses.

### Story 5.1: Implement ILP packet types with OER encoding

As a **Mayor agent**,
I want **ILP packet types (PREPARE, FULFILL, REJECT) implemented with OER encoding**,
So that **I can construct and parse ILP packets according to ILPv4 specification** (NFR-I3).

**Acceptance Criteria:**

**Given** the internal/ilp/types.go file exists
**When** I review the ILP packet type definitions
**Then** structs exist for:
- `ILPPrepare` - contains: Amount, ExpiresAt, ExecutionCondition, Destination, Data
- `ILPFulfill` - contains: FulfillmentCondition, Data
- `ILPReject` - contains: Code, TriggeredBy, Message, Data
**And** all structs follow ILPv4 (RFC 0027) specification

**Given** ILP packet structs are defined
**When** I review the OER (Octet Encoding Rules) implementation
**Then** each struct has methods:
- `MarshalOER() ([]byte, error)` - encodes to OER binary format
- `UnmarshalOER(data []byte) error` - decodes from OER binary format
**And** OER encoding follows RFC 0030 specification
**And** variable-length fields use proper length prefixes

**Given** I create an ILP PREPARE packet
**When** I construct `ILPPrepare{Amount: 875, Destination: "g.relay.crosstown", Data: toonBytes}`
**Then** the struct is properly initialized
**And** ExecutionCondition is a 32-byte hash (SHA-256 of fulfillment)
**And** ExpiresAt is set to current time + 30 seconds
**And** Amount is in base units (per asset scale)

**Given** I encode an ILP packet to OER
**When** I call `packet.MarshalOER()`
**Then** it returns binary data in OER format
**And** the binary follows ILPv4 packet structure:
- Type (1 byte): 0x0C for PREPARE, 0x0D for FULFILL, 0x0E for REJECT
- Length-prefixed fields for all variable data
**And** all numeric fields use proper endianness

**Given** I decode an ILP packet from OER
**When** I call `UnmarshalOER(oerData)`
**Then** it populates the struct fields correctly
**And** it validates packet structure during decoding
**And** it returns error for malformed OER data

**Given** unit tests exist in types_test.go
**When** I run `go test ./internal/ilp/`
**Then** TestILPPacketMarshalUnmarshal passes with various packet types
**And** TestOEREncoding verifies binary format matches spec
**And** test coverage for packet encoding is ≥90%

### Story 5.2: Implement TOON encoding for DVM job events

As a **Mayor agent**,
I want **TOON encoding implementation for DVM job request events (kind:5xxx)**,
So that **I can construct Nostr DVM job events in TOON format** (FR15, NFR-I4).

**Acceptance Criteria:**

**Given** the internal/nostr/encoding.go file exists with stubs
**When** I review the TOON encoding implementation
**Then** it uses `github.com/ALLiDoizCode/toon-go` library
**And** a function `EncodeDVMJobEvent(jobData JobRequest) ([]byte, error)` exists
**And** it follows TOON specification from https://github.com/nicholasgasior/toon

**Given** I want to create a DVM job request event
**When** I provide JobRequest with fields:
- `Kind`: 5000 (DVM job request kind)
- `Content`: job description string
- `Tags`: array of tag arrays (e.g., [["i", "input_data"], ["output", "json"]])
- `PubKey`: Mayor's public key
**Then** the encoder constructs a Nostr event structure
**And** it adds timestamp (created_at field)
**And** it generates event ID (SHA-256 hash of serialized event)

**Given** the Nostr event structure is ready
**When** I call `EncodeDVMJobEvent(jobRequest)`
**Then** it encodes the event to TOON binary format using toon-go
**And** TOON encoding is more compact than JSON (NFR-I4)
**And** the binary output can be directly embedded in ILP packet data
**And** no JSON serialization step is required

**Given** I want to verify TOON encoding
**When** I encode a test job event
**Then** the output is valid TOON binary
**And** decoding the TOON bytes returns the original event structure
**And** the encoding preserves all fields (kind, content, tags, pubkey, created_at, id)

**Given** TOON encoding includes LLM-friendly structure
**When** reviewing the encoded format
**Then** it follows TOON structural regularity guidelines
**And** it supports LLM-native processing (FR28) by Claude Code
**And** the format allows pattern recognition without explicit schema

**Given** unit tests exist in encoding_test.go
**When** I run `go test ./internal/nostr/`
**Then** TestEncodeDVMJobEvent passes with various job types
**And** TestTOONRoundTrip verifies encode/decode consistency
**And** test coverage for TOON encoding is ≥85%

### Story 5.3: Implement job posting via ILP PREPARE packets

As a **Mayor agent**,
I want **to post DVM jobs via ILP PREPARE packets with TOON payloads**,
So that **I can distribute jobs to the crosstown network and pay for relay access** (FR16).

**Acceptance Criteria:**

**Given** ILP packets and TOON encoding exist (Stories 5.1, 5.2)
**When** I review the job posting implementation in internal/crosstown/
**Then** a function `PostJob(jobRequest JobRequest, channelID string) (*JobReceipt, error)` exists
**And** it coordinates ILP packet creation and BTP transmission

**Given** I call PostJob with a job request
**When** the function executes
**Then** the process follows these steps:
1. Encode job request to TOON format
2. Calculate payment amount (bytes * basePricePerByte from SPSP)
3. Create ILP PREPARE packet with TOON data as payload
4. Deduct payment amount from channel balance
5. Send PREPARE packet via BTP client to connector
6. Wait for FULFILL or REJECT response
7. Return JobReceipt with job ID and status

**Given** payment amount is calculated
**When** TOON payload is X bytes
**Then** payment amount = X * basePricePerByte (from SPSP handshake)
**And** channel balance is verified >= payment amount before sending
**And** if insufficient balance, return error: "Channel balance too low"

**Given** ILP PREPARE packet is created
**When** constructing the packet
**Then** it includes:
- Amount: calculated payment amount
- Destination: ILP address from SPSP (e.g., "g.relay.crosstown")
- Data: TOON-encoded job event bytes
- ExecutionCondition: SHA-256 hash of fulfillment secret
- ExpiresAt: current time + 30 seconds
**And** packet is encoded to OER binary format

**Given** the PREPARE packet is sent
**When** transmission completes
**Then** the packet is sent via BTP client (Story 4.2)
**And** request is logged: "Posting job job-5abc123 (875 bytes, 0.00875 M2M)"
**And** the function waits for response (FULFILL or REJECT)

**Given** payment flow completes within latency requirement
**When** measuring end-to-end time
**Then** PREPARE → FULFILL flow completes in <500ms (NFR-P1)
**And** if it takes >500ms, Mayor logs performance warning

**Given** unit tests exist in crosstown_test.go
**When** I run `go test ./internal/crosstown/`
**Then** TestPostJob passes with mocked BTP client
**And** TestPaymentCalculation verifies amount calculation
**And** TestLatencyRequirement verifies <500ms performance
**And** test coverage for job posting is ≥85%

### Story 5.4: Implement ILP response handling (FULFILL/REJECT)

As a **Mayor agent**,
I want **to handle ILP FULFILL and REJECT responses from the connector**,
So that **I know if my job was posted successfully or if payment failed** (FR17).

**Acceptance Criteria:**

**Given** a PREPARE packet was sent (Story 5.3)
**When** the connector responds with FULFILL
**Then** the response is received via BTP client
**And** it is decoded from OER to ILPFulfill struct
**And** FulfillmentCondition is validated (must hash to ExecutionCondition)
**And** if validation passes, job posting is confirmed successful

**Given** FULFILL response is validated
**When** handling the successful payment
**Then** channel balance is updated (deduct payment amount)
**And** updated balance is persisted to git worktree TOML file (Story 4.5)
**And** git commit is created: "Payment: job-5abc123 (0.00875 M2M, ch_abc123)"
**And** success is logged: "✅ Job posted: job-5abc123"
**And** JobReceipt is returned with status: "posted"

**Given** the connector responds with REJECT
**When** the response is received
**Then** it is decoded from OER to ILPReject struct
**And** reject code is examined (e.g., T00=temporary, F00=final)
**And** reject message is extracted for logging

**Given** REJECT response indicates temporary failure (T-codes)
**When** processing the rejection
**Then** channel balance is NOT deducted (payment not completed)
**And** error is logged: "⚠️  Job posting rejected (temporary): <message>"
**And** the error is returned for retry logic (Story 5.6)
**And** JobReceipt is NOT created (payment failed)

**Given** REJECT response indicates final failure (F-codes)
**When** processing the rejection
**Then** channel balance is NOT deducted
**And** error is logged: "❌ Job posting failed (final): <message>"
**And** detailed error is returned to caller
**And** no retry is attempted (final failure)

**Given** response handling includes timeout
**When** no FULFILL/REJECT received within 30 seconds
**Then** the operation times out
**And** error is returned: "Payment timeout: no response from connector"
**And** channel balance is NOT deducted (uncertain state)
**And** operator is advised to check channel state manually

**Given** unit tests exist
**When** I run `go test ./internal/crosstown/`
**Then** TestHandleFulfill passes
**And** TestHandleReject verifies all reject code types
**And** TestPaymentTimeout verifies timeout handling
**And** test coverage for response handling is ≥90%

### Story 5.5: Implement job correlation tracking in git worktree

As a **Mayor agent**,
I want **job correlation mappings stored in git worktree**,
So that **I can route incoming results to the correct callback skills** (FR18).

**Acceptance Criteria:**

**Given** the internal/crosstown/job_tracker.go file exists with stubs
**When** I review the job correlation implementation
**Then** a `JobTracker` struct exists with methods:
- `StoreCorrelation(jobID string, callbackSkill string) error`
- `GetCorrelation(jobID string) (string, error)`
- `RemoveCorrelation(jobID string) error`
- `ListActiveJobs() ([]string, error)`
**And** correlations are persisted in git worktree

**Given** a job is posted successfully (Story 5.3/5.4)
**When** I call `StoreCorrelation("job-5abc123", "build-skill")`
**Then** a correlation file is created: `~/.gt/hooks/mayor-{id}/crosstown/jobs/job-5abc123.toml`
**And** file permissions are 0600 (owner read/write only)
**And** the TOML file contains:
```toml
job_id = "job-5abc123"
callback_skill = "build-skill"
created_at = "2026-02-18T14:23:45Z"
status = "pending"
```

**Given** correlation file is created
**When** the correlation is committed to git
**Then** git commit is created: "Job correlation: job-5abc123 → build-skill"
**And** the commit provides audit trail of job tracking
**And** correlation survives Mayor crashes (NFR-R1)

**Given** I need to retrieve a correlation
**When** I call `GetCorrelation("job-5abc123")`
**Then** it reads the TOML file from git worktree
**And** it parses and returns the callback_skill value
**And** if job ID not found, return error: "No correlation found for job-5abc123"

**Given** a job result is received and processed
**When** I call `RemoveCorrelation("job-5abc123")`
**Then** the correlation file status is updated to "completed"
**And** file is kept (not deleted) for audit trail
**And** git commit is created: "Job completed: job-5abc123"

**Given** I want to list all active jobs
**When** I call `ListActiveJobs()`
**Then** it scans crosstown/jobs/ directory for TOML files
**And** it returns array of job IDs with status="pending"
**And** completed jobs are excluded from the list

**Given** git operations are serialized (NFR-R6)
**When** multiple correlations are stored concurrently
**Then** all git writes are serialized using gastown's locking
**And** no race conditions occur

**Given** unit tests exist in job_tracker_test.go
**When** I run `go test ./internal/crosstown/`
**Then** TestStoreCorrelation passes
**And** TestGetCorrelation verifies retrieval
**And** TestListActiveJobs verifies filtering
**And** test coverage for job tracking is ≥90%

### Story 5.6: Add ILP payment retry logic with exponential backoff

As a **Mayor agent**,
I want **automatic retry of failed ILP payments with exponential backoff**,
So that **transient network errors don't cause permanent job posting failures** (NFR-R4).

**Acceptance Criteria:**

**Given** job posting exists (Story 5.3) with response handling (Story 5.4)
**When** I review the retry logic implementation
**Then** PostJob function includes retry wrapper
**And** retry parameters are configured:
- Max retries: 3
- Initial delay: 1 second
- Backoff multiplier: 2.0
**And** only temporary failures (REJECT with T-codes or network errors) are retried

**Given** a job posting fails with temporary error
**When** ILP REJECT with code T00 (temporary failure) is received
**Then** PostJob automatically retries after 1 second
**And** retry is logged: "Retrying job post (attempt 2/3) after temporary failure"
**And** the same PREPARE packet is resent

**Given** the first retry fails
**When** second temporary failure occurs
**Then** PostJob waits 2 seconds before third attempt
**And** third attempt waits 4 seconds before final attempt
**And** each retry is logged with attempt number

**Given** a retry succeeds
**When** FULFILL is received on attempt 2 or 3
**Then** the job is posted successfully
**And** success is logged: "✅ Job posted on retry attempt 2: job-5abc123"
**And** channel balance is deducted once (for successful attempt)

**Given** all retries are exhausted
**When** 3 attempts all fail with temporary errors
**Then** PostJob returns error: "Job posting failed after 3 retries"
**And** no channel balance is deducted
**And** caller can decide whether to retry manually or abort

**Given** a final failure occurs (REJECT with F-code)
**When** processing the rejection
**Then** NO retry is attempted (final failure is not retryable)
**And** error is immediately returned to caller
**And** retry counter is not incremented

**Given** network errors occur
**When** BTP send fails with network timeout or connection error
**Then** the error is classified as temporary
**And** retry logic is triggered
**And** exponential backoff is applied

**Given** unit tests exist
**When** I run `go test ./internal/crosstown/`
**Then** TestPaymentRetry verifies retry behavior
**And** TestExponentialBackoff verifies delay calculations
**And** TestFinalFailureNoRetry verifies F-codes skip retry
**And** test coverage for retry logic is ≥90%

## Epic 6: Job Results & Earnings

Mayor can publish DVM results, receive payments for completed work, and execute jobs in Polecat worktrees.

### Story 6.1: Implement Polecat job execution in git worktrees

As a **Polecat agent**,
I want **to execute assigned jobs in dedicated git worktrees**,
So that **my work is isolated and tracked through gastown's propulsion principle** (FR24).

**Acceptance Criteria:**

**Given** Mayor receives a kind:5xxx DVM job request from the relay
**When** the job is routed to a Polecat (via skills system, Epic 8)
**Then** Mayor spawns a Polecat in a dedicated git worktree
**And** the worktree path follows gastown pattern: `~/.gt/hooks/pole cat-{id}/`
**And** job request data is made available to Polecat

**Given** Polecat is executing the job
**When** work is performed
**Then** all outputs and state are committed to the Polecat's git worktree
**And** git commits provide audit trail of work performed
**And** worktree isolation prevents conflicts with other Polecats

**Given** Polecat completes the job
**When** work is finished
**Then** Polecat signals completion to Mayor
**And** result data is extracted from Polecat's git worktree
**And** result is prepared for publishing (Story 6.2)

**Given** Polecat execution fails
**When** job cannot be completed
**Then** error details are captured from Polecat
**And** Mayor can construct error result event
**And** failure is logged with job ID and reason

**Given** unit tests exist
**When** I run tests
**Then** Polecat spawning and isolation is verified
**And** worktree creation follows gastown patterns
**And** test coverage for Polecat execution is ≥80%

### Story 6.2: Implement DVM result event construction with job correlation

As a **Mayor agent**,
I want **to construct kind:6xxx DVM result events with proper job correlation**,
So that **requestors can identify which job my result corresponds to** (FR25).

**Acceptance Criteria:**

**Given** Polecat completed a job (Story 6.1)
**When** Mayor constructs the result event
**Then** a kind:6xxx DVM result event is created with fields:
- `Kind`: 6000 (DVM result kind)
- `Content`: result data or error message
- `Tags`: includes correlation tags linking to original job
  - `["e", "<original_job_event_id>", "reply"]` (job correlation)
  - `["p", "<requestor_pubkey>"]` (requestor reference)
  - `["status", "success"]` or `["status", "error"]`
- `PubKey`: Mayor's public key

**Given** job correlation exists (from Story 5.5)
**When** constructing the result
**Then** Mayor retrieves the original job event ID from correlation state
**And** includes it in the "e" tag for proper threading
**And** result can be matched to request by relay and requestor

**Given** result includes payment request
**When** Mayor wants to earn M2M tokens for the work
**Then** result event includes payment request tags (if applicable)
**And** amount requested is based on work performed
**And** payment details follow DVM payment conventions

**Given** result event is constructed
**When** ready for encoding
**Then** the event structure is complete and valid
**And** it can be encoded to TOON format (via Story 5.2 encoding)
**And** encoded result is ready for ILP publishing (Story 6.3)

**Given** unit tests exist in crosstown_test.go
**When** I run `go test ./internal/crosstown/`
**Then** TestConstructResultEvent passes
**And** TestJobCorrelationTags verifies proper tagging
**And** test coverage for result construction is ≥85%

### Story 6.3: Implement result publishing and payment earning

As a **Mayor agent**,
I want **to publish DVM results via ILP and earn M2M token payments**,
So that **I'm compensated for work performed by my Polecats** (FR26).

**Acceptance Criteria:**

**Given** a DVM result event is constructed (Story 6.2)
**When** I call `PublishResult(resultEvent ResultEvent, channelID string)`
**Then** the process follows these steps:
1. Encode result event to TOON format
2. Create ILP PREPARE packet with TOON result as payload
3. Send PREPARE via BTP to connector
4. Wait for FULFILL response (payment received)
5. Update channel balance (credit payment amount)
6. Persist updated balance to git worktree
7. Return PublishReceipt with earnings amount

**Given** result is published successfully
**When** FULFILL response is received
**Then** the fulfillment includes payment amount earned
**And** channel balance is increased by payment amount
**And** balance update is committed to git: "Earned 0.01 M2M for result: job-5abc123"
**And** success is logged: "✅ Result published, earned 0.01 M2M"

**Given** result publishing fails
**When** REJECT response is received or timeout occurs
**Then** error is logged with failure reason
**And** result is not marked as published
**And** operator can retry publishing manually if needed

**Given** payment earning is tracked
**When** earnings are received
**Then** earnings are logged to payment_log.jsonl (Story 10.2)
**And** payment statistics are updated (total earned)
**And** earnings are visible in dashboard (Story 10.1)

**Given** unit tests exist
**When** I run `go test ./internal/crosstown/`
**Then** TestPublishResult passes
**And** TestEarningsTracking verifies balance increases
**And** test coverage for result publishing is ≥85%

### Story 6.4: Add end-to-end job execution and payment flow

As an **operator**,
I want **end-to-end job execution from receipt to payment**,
So that **my Mayor can both consume and provide DVM services** (FR24, FR25, FR26).

**Acceptance Criteria:**

**Given** all previous stories in Epic 6 are complete
**When** I review the complete job execution flow
**Then** the integration works end-to-end:
1. Mayor receives kind:5xxx job request from relay
2. Job is routed to appropriate skill (Epic 8)
3. Polecat executes job in isolated worktree
4. Mayor constructs kind:6xxx result with correlation
5. Result is published via ILP, earning payment
6. Channel balance is updated with earnings
7. Job correlation is marked completed

**Given** Mayor processes a complete job cycle
**When** end-to-end flow executes
**Then** all state transitions are logged
**And** git worktree contains audit trail of:
  - Job correlation created
  - Polecat work performed
  - Result published
  - Payment earned
  - Job correlation completed

**Given** integration tests exist
**When** I run E2E tests
**Then** full job execution flow is validated
**And** payment flows (both spending and earning) work correctly
**And** test coverage for integration is ≥75%

## Epic 7: Job Routing & Correlation

Mayor can route incoming job results to appropriate callback skills using git-backed correlation state.

### Story 7.1: Implement result event subscription and reception

As a **Mayor agent**,
I want **to subscribe to kind:6xxx result events filtered by my posted job IDs**,
So that **I receive only results relevant to jobs I posted** (FR19).

**Acceptance Criteria:**

**Given** Mayor has posted jobs (Epic 5) with correlations tracked (Story 5.5)
**When** Mayor wants to receive results
**Then** it subscribes to relay with filter:
- `Kinds`: [6000, 6001, 6002] (DVM result kinds)
- `Tags`: filter by "e" tag matching posted job event IDs
**And** subscription uses relay client from Epic 3

**Given** subscription is active
**When** relay sends kind:6xxx events
**Then** Mayor receives TOON-encoded result events
**And** events are passed to result routing logic (Story 7.2)
**And** only results matching posted job IDs are processed

**Given** result event is received
**When** parsing the event tags
**Then** Mayor extracts the "e" tag (job event ID reference)
**And** uses it to look up correlation mapping
**And** prepares for routing to callback skill

**Given** unit tests exist
**When** I run `go test ./internal/crosstown/`
**Then** TestResultSubscription verifies filter construction
**And** TestResultReception verifies event processing
**And** test coverage for result subscription is ≥85%

### Story 7.2: Implement result routing to callback skills

As a **Mayor agent**,
I want **to route received results to the correct callback skills**,
So that **each skill gets the results for jobs it requested** (FR20).

**Acceptance Criteria:**

**Given** a kind:6xxx result event is received (Story 7.1)
**When** Mayor processes the result
**Then** it extracts job event ID from "e" tag
**And** looks up correlation: `GetCorrelation(jobEventID)`
**And** retrieves callback_skill name (e.g., "build-skill")

**Given** callback skill is identified
**When** routing the result
**Then** Mayor invokes the callback skill via Claude Code skills system (Epic 8)
**And** passes the raw TOON result bytes to the skill
**And** skill can process the result and take appropriate action

**Given** result routing succeeds
**When** skill receives the result
**Then** correlation is marked as completed: `RemoveCorrelation(jobID)`
**And** git commit: "Job completed: job-5abc123 (routed to build-skill)"
**And** success is logged: "✅ Result routed to build-skill"

**Given** correlation not found
**When** no matching job ID exists
**Then** Mayor logs warning: "Received result for unknown job: <job_id>"
**And** result is logged to unmatched results file for debugging
**And** no skill invocation occurs

**Given** skill invocation fails
**When** callback skill errors during processing
**Then** error is logged with skill name and job ID
**And** correlation remains active (not completed)
**And** operator can investigate and retry manually

**Given** unit tests exist
**When** I run `go test ./internal/crosstown/`
**Then** TestResultRouting passes
**And** TestCallbackSkillInvocation verifies skill integration
**And** test coverage for routing is ≥85%

### Story 7.3: Add result routing observability and error handling

As an **operator**,
I want **visibility into result routing and error handling**,
So that **I can monitor and debug job correlation issues**.

**Acceptance Criteria:**

**Given** result routing is active (Stories 7.1, 7.2)
**When** results are being processed
**Then** all routing decisions are logged:
- "Received result for job-5abc123"
- "Routing to callback skill: build-skill"
- "Skill invocation successful"

**Given** routing metrics are tracked
**When** I query routing statistics
**Then** Mayor tracks:
- Total results received
- Successfully routed results
- Unmatched results (no correlation)
- Failed skill invocations
**And** statistics are available via CLI command

**Given** unmatched results accumulate
**When** viewing unmatched results log
**Then** file exists: `crosstown/unmatched_results.jsonl`
**And** each line contains: timestamp, job_id, result_event
**And** operator can investigate why correlation is missing

**Given** unit tests exist
**When** I run tests
**Then** routing observability is verified
**And** error handling paths are tested
**And** test coverage for observability is ≥80%

## Epic 8: TOON Event Processing & Skills Integration

Claude Code and skills can process raw TOON events natively without JSON conversion, with automatic skill discovery.

### Story 8.1: Implement zero-parsing TOON pass-through to Claude Code

As a **Mayor agent**,
I want **to pass raw TOON byte arrays to Claude Code with zero parsing in Go**,
So that **TOON events can be processed directly by LLM without serialization overhead** (FR27).

**Acceptance Criteria:**

**Given** Mayor receives TOON-encoded events from relay (Epic 3)
**When** event needs to be processed
**Then** Go networking layer does NOT decode TOON to JSON
**And** raw TOON bytes are passed directly to Claude Code
**And** no schema-based parsing occurs in Go code

**Given** TOON bytes are passed to Claude Code
**When** invoking skill with TOON payload
**Then** Mayor calls Claude Code skill invocation interface
**And** passes: `skillName`, `rawTOONBytes []byte`
**And** skill interface accepts raw bytes without type constraints

**Given** Claude Code receives raw TOON
**When** processing the event
**Then** Claude Code uses LLM-native TOON interpretation (FR28)
**And** pattern recognition on binary byte sequences
**And** structural regularity enables field extraction
**And** fewer tokens required compared to JSON parsing

**Given** unit tests exist
**When** I run tests
**Then** zero-parsing flow is verified
**And** no JSON encoding/decoding occurs in Go layer
**And** test coverage for pass-through is ≥85%

### Story 8.2: Implement LLM-native TOON kind extraction

As **Claude Code**,
I want **to extract event kind field from TOON payload for routing**,
So that **events can be routed to appropriate skills without full parsing** (FR29).

**Acceptance Criteria:**

**Given** Claude Code receives raw TOON bytes
**When** kind extraction is needed for routing
**Then** Claude Code applies pattern recognition to TOON structure
**And** identifies kind field offset in binary format
**And** extracts kind value (e.g., 5000, 6000) in <1ms (NFR-P2)

**Given** kind is extracted
**When** routing decision is made
**Then** kind value determines skill category:
- 5xxx → DVM job request handlers
- 6xxx → DVM result handlers
**And** appropriate skill is selected based on kind

**Given** TOON structure allows efficient extraction
**When** measuring performance
**Then** kind extraction completes in <1ms per event
**And** no explicit schema definitions are required
**And** TOON structural regularity enables fast parsing

**Given** extraction is validated
**When** testing with various TOON events
**Then** kind is correctly extracted from all event types
**And** extraction accuracy is 100%
**And** performance meets <1ms requirement

### Story 8.3: Implement automatic skill discovery and routing

As a **skills developer**,
I want **automatic skill discovery from .claude/skills/ directory**,
So that **I can add new event handlers without modifying Mayor core** (FR55, FR56, FR58).

**Acceptance Criteria:**

**Given** the .claude/skills/ directory exists
**When** Mayor starts up
**Then** Claude Code scans .claude/skills/ for skill files
**And** each skill declares supported event kinds in frontmatter
**And** skill registration happens automatically (zero configuration)

**Given** a skill declares supported kinds
**When** reviewing skill file frontmatter
**Then** it includes: `supportedKinds: [5000, 5001]`
**And** Claude Code builds routing map: kind → skill name
**And** routing is updated dynamically when skills are added/removed

**Given** an event with kind:5000 is received
**When** Claude Code routes the event
**Then** it looks up kind:5000 in routing map
**And** invokes the registered skill (e.g., "build-job-handler")
**And** passes raw TOON payload to skill (FR57)

**Given** a new skill is added
**When** developer creates `.claude/skills/new-handler.md`
**Then** skill is automatically discovered on next event
**And** no Mayor restart required
**And** no changes to Mayor core code (FR58)

**Given** skill routing is tested
**When** running integration tests
**Then** automatic discovery works correctly
**And** kind-based routing is accurate
**And** test coverage for skill integration is ≥80%

### Story 8.4: Implement skill event handling with raw TOON

As a **skill developer**,
I want **skills to receive and process raw TOON payloads**,
So that **event handling is efficient and LLM-native** (FR31, FR32, FR57).

**Acceptance Criteria:**

**Given** a skill is invoked with TOON payload
**When** skill receives the event
**Then** it gets: `eventKind: number`, `toonPayload: bytes`
**And** skill can process TOON directly using LLM interpretation
**And** no JSON conversion is required

**Given** skill handles kind:5xxx (DVM job request)
**When** processing job request
**Then** skill extracts job parameters from TOON
**And** interprets job instructions
**And** initiates Polecat execution (Story 6.1)
**And** returns acknowledgment or error

**Given** skill handles kind:6xxx (DVM result)
**When** processing result event
**Then** skill extracts result data from TOON
**And** processes outcome (success/failure)
**And** triggers follow-up actions if needed
**And** marks job as completed

**Given** skill error handling exists
**When** TOON processing fails
**Then** skill returns detailed error to Mayor
**And** error is logged with event kind and skill name
**And** event is saved to failed events log for debugging

**Given** unit tests exist for skills
**When** testing event handlers
**Then** TOON payload processing is verified
**And** skill behavior for both job kinds (5xxx, 6xxx) is tested
**And** test coverage for skill handlers is ≥75%

### Story 8.5: Add skills extensibility and documentation

As a **skills developer**,
I want **clear documentation and examples for extending Mayor with new skills**,
So that **I can easily add custom DVM event handlers**.

**Acceptance Criteria:**

**Given** skills system is implemented (Stories 8.1-8.4)
**When** reviewing documentation
**Then** guide exists: `docs/crosstown-skills-guide.md`
**And** it explains:
- Skill file structure
- supportedKinds declaration
- TOON payload handling
- Callback skill registration
- Testing skills

**Given** example skills are provided
**When** reviewing .claude/skills/ directory
**Then** example skills exist:
- `example-job-handler.md` (kind:5xxx handler)
- `example-result-handler.md` (kind:6xxx callback)
**And** examples include commented TOON processing logic

**Given** skills can be tested independently
**When** developer creates a new skill
**Then** test harness allows testing with sample TOON events
**And** debugging output shows TOON interpretation
**And** skill can be validated before deployment

## Epic 9: Advanced Job Orchestration

Mayor can execute sequential job chains, handle timeouts gracefully, and recover in-flight jobs after crashes.

### Story 9.1: Implement sequential job chains

As a **Mayor agent**,
I want **to execute sequential job chains where Job A completion triggers Job B**,
So that **complex multi-step workflows can be automated** (FR21).

**Acceptance Criteria:**

**Given** the internal/crosstown/ package has job chaining support
**When** I define a job chain
**Then** chain is defined as: `JobChain{jobs: [JobA, JobB, JobC], onSuccess: nextJob}`
**And** each job includes success criteria and next job trigger

**Given** a job chain is initiated
**When** Job A is posted (Epic 5)
**Then** correlation includes: `nextJobInChain: "JobB"`
**And** chain state is persisted to git worktree
**And** Mayor waits for Job A result

**Given** Job A completes successfully
**When** result is received (Epic 7)
**Then** Mayor checks correlation for `nextJobInChain`
**And** automatically posts Job B with context from Job A result
**And** Job B correlation links back to chain ID
**And** chain continues until all jobs complete

**Given** a job in the chain fails
**When** error result is received
**Then** Mayor halts the chain execution
**And** failure is logged with chain ID and failed job
**And** subsequent jobs are NOT posted
**And** operator is notified of chain failure

**Given** unit tests exist
**When** I run tests
**Then** sequential chain execution is verified
**And** failure handling stops chain correctly
**And** test coverage for job chains is ≥85%

### Story 9.2: Implement job timeout handling

As a **Mayor agent**,
I want **to handle job timeouts gracefully when no result is received**,
So that **Mayor doesn't block indefinitely waiting for results** (FR23, NFR-R5).

**Acceptance Criteria:**

**Given** a job is posted (Epic 5)
**When** the job correlation is created
**Then** a timeout timestamp is recorded (default: 300 seconds from now)
**And** timeout is configurable via GASTOWN_JOB_TIMEOUT_SECONDS

**Given** Mayor monitors active jobs
**When** periodic timeout check runs (every 60 seconds)
**Then** it scans all active job correlations
**And** compares current time to timeout timestamp
**And** identifies jobs that have exceeded timeout

**Given** a job has timed out
**When** timeout is detected
**Then** job correlation status is updated to "timeout"
**And** git commit: "Job timeout: job-5abc123 (no result after 300s)"
**And** warning is logged: "⚠️  Job timed out: job-5abc123"
**And** callback skill is notified of timeout (if registered)

**Given** timeout notifications are sent
**When** callback skill is invoked
**Then** it receives: `jobID`, `status: "timeout"`, `error: "No result received"`
**And** skill can handle timeout (retry, abort, alert operator)
**And** timeout handling is skill-specific

**Given** timed-out jobs don't block Mayor
**When** timeout occurs
**Then** Mayor continues normal operations
**And** no indefinite waiting occurs (NFR-R5)
**And** other jobs are processed normally

**Given** unit tests exist
**When** I run tests
**Then** timeout detection logic is verified
**And** timeout handling is tested
**And** test coverage for timeouts is ≥85%

### Story 9.3: Implement crash recovery for in-flight jobs

As a **Mayor agent**,
I want **to resume in-flight job chains after Mayor crashes**,
So that **work is not lost and chains continue from interruption point** (FR22, NFR-R1).

**Acceptance Criteria:**

**Given** Mayor crashes during job execution
**When** Mayor restarts
**Then** it reads all job correlations from git worktree
**And** identifies jobs with status: "pending" or "in-chain"
**And** reconstructs in-flight job state

**Given** in-flight jobs are identified
**When** resuming from crash
**Then** Mayor checks each job's timeout status
**And** jobs not yet timed out are kept active
**And** subscriptions are re-established for pending results
**And** job chains are resumed from last completed step

**Given** a job chain was interrupted mid-execution
**When** Job A completed but Job B wasn't posted yet
**Then** Mayor detects Job A completion from correlation state
**And** automatically posts Job B to continue the chain
**And** chain execution resumes seamlessly

**Given** crash recovery is logged
**When** Mayor resumes
**Then** it logs: "Recovering X in-flight jobs after restart"
**And** each resumed job is logged individually
**And** git audit trail shows crash and recovery events

**Given** integration tests exist
**When** testing crash recovery
**Then** simulated crashes during job execution are tested
**And** recovery correctly resumes job chains
**And** no data loss occurs (NFR-R1)
**And** test coverage for recovery is ≥80%

### Story 9.4: Add job orchestration monitoring and controls

As an **operator**,
I want **visibility and control over job orchestration**,
So that **I can monitor chains, cancel jobs, and troubleshoot issues**.

**Acceptance Criteria:**

**Given** job orchestration is running
**When** I run `gt crosstown jobs list`
**Then** it displays all active jobs:
```
JOB ID         STATUS     CHAIN      TIMEOUT IN    CALLBACK SKILL
job-5abc123    Pending    chain-1    245s          build-skill
job-6def456    In Chain   chain-1    180s          deploy-skill
```

**Given** I want to view chain details
**When** I run `gt crosstown jobs chain <chain-id>`
**Then** it shows chain structure:
- All jobs in chain
- Current job status
- Completed jobs
- Remaining jobs
- Chain success/failure status

**Given** I want to cancel a job
**When** I run `gt crosstown jobs cancel <job-id>`
**Then** correlation status is set to "cancelled"
**And** job is removed from active monitoring
**And** if part of chain, chain execution stops
**And** cancellation is logged to git

**Given** observability metrics exist
**When** viewing job statistics
**Then** Mayor tracks:
- Total jobs posted
- Jobs completed successfully
- Jobs failed
- Jobs timed out
- Average job completion time
**And** metrics are available via CLI

## Epic 10: Observability & Audit Trail

Operators can monitor crosstown activity via dashboard, audit all payments in git, and export payment history.

### Story 10.1: Implement Bubbletea crosstown dashboard panel

As an **operator**,
I want **a crosstown network panel in Mayor's Bubbletea dashboard**,
So that **I can monitor connections, channels, and activity in real-time** (FR39, FR46, NFR-I6).

**Acceptance Criteria:**

**Given** internal/mayor/dashboard.go exists
**When** I review crosstown dashboard integration
**Then** a new panel exists: `dashboard_crosstown.go`
**And** it follows Bubbletea MVU (Model-View-Update) pattern (NFR-I6)
**And** state is immutable, updates are pure functions

**Given** Mayor is running with dashboard visible
**When** I run `gt mayor attach`
**Then** crosstown panel is displayed with sections:
- Connection Status (relay + connector)
- Payment Channel (balance, status, network)
- Subscriptions (active event kinds)
- Recent Activity (last 5 events/payments)

**Given** dashboard displays connection status
**When** viewing the relay section
**Then** it shows:
- "Status: ✅ Connected" or "❌ Disconnected"
- "Relay: ws://relay.crosstown.network"
- "Subscriptions: 2 active (kinds: 5xxx, 6xxx)"

**Given** dashboard displays channel info
**When** viewing the payment channel section
**Then** it shows:
- "Balance: 35.42 M2M tokens"
- "Chain: Base Sepolia (84532)"
- "Channel ID: ch_abc123"
- "Status: Open"
**And** balance updates in real-time (<100ms per NFR-P4)

**Given** dashboard shows recent activity
**When** events or payments occur
**Then** activity log displays:
- "14:23:45 📤 Posted job-5abc123 (875 bytes, 0.00875 M2M)"
- "14:24:32 📥 Received result from Mayor-Beta"
- "14:24:33 ✅ Job completed successfully"
**And** timestamps are in local time

**Given** dashboard updates reflect state changes
**When** connection drops or balance changes
**Then** dashboard updates within 100ms (NFR-P4)
**And** no visual lag or stale data
**And** Bubbletea message passing ensures reactivity

**Given** dashboard includes network identification
**When** viewing active network
**Then** it prominently displays: "Base Sepolia Testnet" (NFR-S8)
**And** color coding: yellow for testnet, red for mainnet
**And** prevents deployment confusion

**Given** unit tests exist in dashboard_crosstown_test.go
**When** I run tests
**Then** MVU pattern is verified
**And** state updates are tested
**And** test coverage for dashboard is ≥75%

### Story 10.2: Implement git audit trail for all payments

As an **operator**,
I want **every ILP payment logged to git with a commit**,
So that **I have a tamper-evident audit trail of all crosstown transactions** (FR40, NFR-S6, NFR-P6).

**Acceptance Criteria:**

**Given** a payment occurs (job posting or result publishing)
**When** ILP FULFILL is received
**Then** payment details are committed to git asynchronously
**And** commit message includes: job ID, amount, bytes, status
**And** example: "Payment: job-5abc123 (-0.00875 M2M, 875 bytes, ch_abc123)"
**And** earnings use positive amounts: "+0.01 M2M"

**Given** git commits are asynchronous (NFR-P6)
**When** payment is processed
**Then** FULFILL response is NOT blocked by git operations
**And** git commit happens in background goroutine
**And** payment latency remains <500ms (NFR-P1)

**Given** git provides tamper-evident audit (NFR-S6)
**When** reviewing payment history
**Then** git log shows complete chronological payment trail
**And** commit hashes provide cryptographic verification
**And** any tampering is detectable via git integrity checks

**Given** payment commits include transaction data
**When** examining a commit
**Then** it includes:
- Changed file: channel TOML (balance update)
- Commit message: payment details
- Timestamp: when payment occurred
- Author: gastown/Mayor

**Given** unit tests exist
**When** I run tests
**Then** asynchronous git commits are verified
**And** commit message format is validated
**And** test coverage for audit logging is ≥85%

### Story 10.3: Implement structured payment logging to JSONL

As an **operator**,
I want **structured payment data logged to append-only JSONL file with rotation**,
So that **I can query and analyze payment history programmatically** (FR41, NFR-SC5).

**Acceptance Criteria:**

**Given** a payment occurs
**When** logging to structured file
**Then** payment is appended to: `~/.gt/hooks/mayor-{id}/crosstown/payment_log.jsonl`
**And** each line is a complete JSON object
**And** file is append-only (no in-place edits)

**Given** JSONL entry is written
**When** examining the log entry
**Then** it contains fields:
```json
{
  "timestamp": "2026-02-18T14:23:45Z",
  "job_id": "job-5abc123",
  "type": "payment_sent",
  "amount_m2m": "-0.00875",
  "bytes": 875,
  "channel_id": "ch_abc123",
  "status": "success"
}
```

**Given** log rotation is configured (NFR-SC5)
**When** daily rollover occurs
**Then** current log is renamed: `payment_log_2026-02-18.jsonl`
**And** new log file is created: `payment_log.jsonl`
**And** 30 days of history are retained
**And** logs older than 30 days are automatically deleted

**Given** log rotation doesn't impact availability
**When** rotation happens
**Then** payments continue logging without interruption (NFR-SC5)
**And** no payments are lost during rotation
**And** rotation is logged to Mayor output

**Given** unit tests exist
**When** I run tests
**Then** JSONL format is validated
**And** log rotation is tested
**And** test coverage for logging is ≥85%

### Story 10.4: Implement payment history export to CSV/JSON

As an **operator**,
I want **to export payment history to CSV and JSON with filtering**,
So that **I can analyze payments in spreadsheets or external tools** (FR42, FR50).

**Acceptance Criteria:**

**Given** payment logs exist (Story 10.3)
**When** I run `gt crosstown export-payments --format csv`
**Then** it reads payment_log.jsonl files
**And** exports to CSV with columns:
- Timestamp, Job ID, Type, Amount (M2M), Bytes, Channel ID, Status
**And** CSV includes header row

**Given** JSON export is requested
**When** I run `gt crosstown export-payments --format json`
**Then** output is a JSON array of payment objects
**And** each object matches JSONL structure
**And** JSON is pretty-printed for readability

**Given** date filtering is supported
**When** I run `gt crosstown export-payments --since 2026-02-15 --until 2026-02-18`
**Then** only payments within date range are exported
**And** date parsing handles multiple formats (YYYY-MM-DD, RFC3339)
**And** filtering is efficient (doesn't load all logs)

**Given** export can filter by type
**When** I run `gt crosstown export-payments --type payment_sent`
**Then** only outgoing payments are exported
**And** earnings (`payment_received`) are excluded
**And** filtering combines with date range

**Given** export outputs to file or stdout
**When** I run `gt crosstown export-payments --output payments.csv`
**Then** data is written to specified file
**And** without `--output`, data goes to stdout
**And** stdout output can be piped to other tools

**Given** unit tests exist
**When** I run tests
**Then** export formats are validated
**And** filtering logic is tested
**And** test coverage for export is ≥85%

### Story 10.5: Implement payment statistics and reporting

As an **operator**,
I want **payment statistics and summary reports**,
So that **I can understand my crosstown spending and earnings patterns** (FR43).

**Acceptance Criteria:**

**Given** payment history exists
**When** I run `gt crosstown stats`
**Then** it displays summary statistics:
```
Crosstown Payment Statistics
============================
Total Payments Sent: 42 (0.3675 M2M)
Total Earnings:      18 (0.18 M2M)
Net Balance:         -0.1875 M2M

Average Cost Per Event: 0.00875 M2M
Average Earnings Per Result: 0.01 M2M
```

**Given** statistics can be time-bounded
**When** I run `gt crosstown stats --since 2026-02-15`
**Then** statistics cover only specified date range
**And** report title indicates: "Statistics (since 2026-02-15)"

**Given** channel-specific stats are available
**When** I run `gt crosstown stats --channel ch_abc123`
**Then** statistics show only payments for that channel
**And** useful for multi-channel deployments

**Given** stats support output formats
**When** I run `gt crosstown stats --format json`
**Then** statistics are output as JSON object
**And** machine-readable for monitoring systems
**And** can be integrated with dashboards

**Given** unit tests exist
**When** I run tests
**Then** statistics calculations are verified
**And** filtering and formatting are tested
**And** test coverage for stats is ≥80%

### Story 10.6: Implement channel state change logging

As an **operator**,
I want **payment channel open/close events logged with transaction hashes**,
So that **I can audit channel lifecycle and on-chain operations** (FR44).

**Acceptance Criteria:**

**Given** a payment channel is opened (Epic 4)
**When** the on-chain transaction confirms
**Then** event is logged to payment_log.jsonl:
```json
{
  "timestamp": "2026-02-18T14:00:00Z",
  "event_type": "channel_opened",
  "channel_id": "ch_abc123",
  "amount_m2m": "50.00",
  "tx_hash": "0x7f3a2b...",
  "settlement_chain": "evm:base:84532"
}
```

**Given** a payment channel is closed (Epic 4)
**When** the close transaction confirms
**Then** event is logged to payment_log.jsonl:
```json
{
  "timestamp": "2026-02-18T16:30:00Z",
  "event_type": "channel_closed",
  "channel_id": "ch_abc123",
  "final_balance_m2m": "35.42",
  "tx_hash": "0xabc123...",
  "settlement_chain": "evm:base:84532"
}
```

**Given** channel events are included in git audit
**When** channel state changes
**Then** git commit includes channel TOML update
**And** commit message references transaction hash
**And** example: "Payment channel closed: ch_abc123 (tx: 0xabc123...)"

**Given** on-chain verification is possible
**When** reviewing channel logs
**Then** tx_hash allows blockchain verification
**And** operator can verify state on Base Sepolia explorer
**And** audit trail links git commits to on-chain transactions

**Given** unit tests exist
**When** I run tests
**Then** channel event logging is verified
**And** transaction hash inclusion is tested
**And** test coverage for channel logging is ≥85%

## Epic 11: CLI Operations & Automation

Operators can manage all crosstown operations via CLI with multiple output formats and automation support.

### Story 11.1: Implement crosstown configuration management

As an **operator**,
I want **CLI commands to manage crosstown configuration**,
So that **I can configure relay, connector, and settings via command line** (FR34, FR37).

**Acceptance Criteria:**

**Given** the gt crosstown command group exists
**When** I run `gt crosstown config set <key> <value>`
**Then** configuration is saved to ~/.gt/crosstown.toml
**And** supported keys include:
- relay_url
- connector_url
- auto_reconnect (true/false)
- job_timeout_seconds
- balance_warning_threshold_m2m

**Given** I run `gt crosstown config get <key>`
**When** retrieving configuration
**Then** it displays current value from config file
**And** shows source: "config file", "env var", or "default"
**And** respects priority: CLI flags > env vars > config file > defaults (FR37)

**Given** I run `gt crosstown config list`
**When** viewing all configuration
**Then** it displays all settings with values and sources:
```
relay_url=ws://relay.crosstown.network (config file)
connector_url=https://connector.crosstown.network (env var)
job_timeout_seconds=300 (default)
```

**Given** configuration validation exists
**When** I set an invalid value
**Then** validation error is returned
**And** example: "Error: relay_url must start with ws:// or wss://"
**And** config file is not modified

**Given** unit tests exist
**When** I run tests
**Then** config management is verified
**And** priority resolution is tested (FR37)
**And** test coverage for config commands is ≥85%

### Story 11.2: Implement multi-format output for all CLI commands

As an **operator**,
I want **all CLI commands to support multiple output formats**,
So that **I can use human-readable or machine-readable output as needed** (FR51).

**Acceptance Criteria:**

**Given** any gt crosstown command exists
**When** I add `--format text` (default)
**Then** output is human-readable with formatting
**And** includes headers, alignment, colors (if terminal)

**Given** I add `--format json`
**When** running any command
**Then** output is valid JSON
**And** suitable for parsing by scripts or tools
**And** all data is included in JSON structure

**Given** I add `--format csv`
**When** running list commands
**Then** output is valid CSV with header row
**And** suitable for importing to spreadsheets
**And** values are properly escaped/quoted

**Given** format applies to all commands
**When** reviewing command implementation
**Then** wallet, channel, relay, jobs commands all support --format
**And** stats and export commands support --format
**And** consistent flag usage across all commands (FR51)

**Given** unit tests exist for each format
**When** I run tests
**Then** all three formats are validated
**And** JSON is parseable
**And** CSV is valid
**And** test coverage for formatting is ≥85%

### Story 11.3: Implement non-interactive mode for automation

As an **operator**,
I want **non-interactive mode with --yes flag for automation**,
So that **I can script crosstown operations without prompts** (FR52, FR54).

**Acceptance Criteria:**

**Given** commands that require confirmation exist
**When** I run with `--yes` or `-y` flag
**Then** all confirmation prompts are skipped
**And** default action is taken automatically
**And** example: `gt crosstown channel close ch_abc123 --yes`

**Given** automation via environment variables
**When** all config is set via env vars
**Then** commands run without reading config files
**And** useful for containerized/CI environments (FR54)
**And** example: `GASTOWN_RELAY_URL=... gt crosstown relay connect --yes`

**Given** exit codes indicate success/failure
**When** commands complete
**Then** exit code 0 for success
**And** exit code 1 for errors
**And** scripting can check $? for status

**Given** quiet mode reduces output
**When** I run with `--quiet` or `-q` flag
**Then** only errors are printed to stderr
**And** normal output is suppressed
**And** useful for cron jobs or background scripts

**Given** unit tests exist
**When** I run tests
**Then** --yes flag behavior is verified
**And** exit codes are tested
**And** test coverage for automation is ≥80%

### Story 11.4: Implement shell completion scripts

As an **operator**,
I want **shell completion for bash, zsh, fish, and PowerShell**,
So that **I can autocomplete crosstown commands and flags** (FR53).

**Acceptance Criteria:**

**Given** Cobra command framework supports completion
**When** I run `gt completion bash`
**Then** bash completion script is generated
**And** script can be sourced in .bashrc

**Given** completion is installed
**When** I type `gt crosstown ch<TAB>`
**Then** it autocompletes to `gt crosstown channel`
**And** subcommands are suggested

**Given** I type `gt crosstown channel <TAB>`
**When** requesting subcommand suggestions
**Then** it shows: open, status, close, list, reconstruct
**And** descriptions are shown (if supported by shell)

**Given** flag completion works
**When** I type `gt crosstown channel open --<TAB>`
**Then** it suggests: --connector, --amount, --format, --yes
**And** flag descriptions are shown

**Given** all shells are supported
**When** I run completion commands
**Then** `gt completion bash|zsh|fish|powershell` all work
**And** generated scripts are valid for each shell
**And** installation instructions are provided

**Given** unit tests exist
**When** I run tests
**Then** completion generation is verified
**And** test coverage for completion is ≥75%

## Epic 12: Production Readiness & Testing

Development team has comprehensive test coverage (≥50%), E2E Docker tests, and production deployment configuration.

### Story 12.1: Achieve ≥50% unit test coverage

As a **development team**,
I want **unit test coverage of at least 50% across all crosstown packages**,
So that **code quality is maintained and regressions are prevented**.

**Acceptance Criteria:**

**Given** all crosstown packages have test files
**When** I run `make test-crosstown`
**Then** all unit tests pass
**And** coverage report is generated

**Given** coverage is measured
**When** I run `go test -cover ./internal/crosstown/... ./internal/ilp/... ./internal/nostr/...`
**Then** overall coverage is ≥50%
**And** individual packages meet minimum thresholds:
- internal/ilp/: ≥85%
- internal/nostr/: ≥85%
- internal/crosstown/: ≥85%

**Given** coverage includes critical paths
**When** reviewing coverage report
**Then** payment flows are 100% covered
**And** security-critical code (encryption, validation) is 100% covered
**And** error handling paths are well-covered

**Given** CI enforces coverage
**When** pull requests are submitted
**Then** CI fails if coverage drops below 50%
**And** coverage report is posted as PR comment

### Story 12.2: Implement E2E tests in Docker containers

As a **development team**,
I want **E2E tests that run in Docker containers**,
So that **full crosstown flow is validated before release**.

**Acceptance Criteria:**

**Given** docker-compose.test.yml exists
**When** I run `make test-e2e-container`
**Then** Docker containers are started:
- gastown (with Mayor)
- crosstown-relay (mock relay)
- crosstown-connector (mock connector)
- base-sepolia-node (mock blockchain)

**Given** E2E test suite runs
**When** containers are ready
**Then** tests execute full workflows:
1. Wallet setup and encryption
2. Payment channel opening
3. Relay connection
4. Job posting and payment
5. Result publishing and earning
6. Channel closing

**Given** tests validate end-to-end flow
**When** reviewing test results
**Then** all major features are tested
**And** tests verify NFR compliance (latency, security)
**And** test coverage for integration is ≥75%

**Given** E2E tests run in CI
**When** pull requests are submitted
**Then** E2E suite runs automatically
**And** failures block merging
**And** test logs are available for debugging

### Story 12.3: Add production deployment configuration

As a **deployment team**,
I want **production-ready deployment configuration**,
So that **crosstown can be deployed to production environments**.

**Acceptance Criteria:**

**Given** deployment guides exist
**When** reviewing docs/deployment/
**Then** guides cover:
- Production environment variables
- TLS certificate setup
- Mainnet configuration
- Monitoring integration
- Backup procedures

**Given** mainnet safeguards exist
**When** deploying to mainnet
**Then** explicit `--enable-mainnet` flag is required
**And** testnet configs don't work on mainnet
**And** dashboard shows "MAINNET" prominently (red)

**Given** monitoring integration is documented
**When** setting up observability
**Then** guides explain:
- Payment log parsing for monitoring
- Alert thresholds
- Dashboard integration
- Metrics export

### Story 12.4: Add golangci-lint configuration and CI integration

As a **development team**,
I want **linting enforced via golangci-lint in CI**,
So that **code quality standards are maintained**.

**Acceptance Criteria:**

**Given** .golangci.yml configuration exists
**When** I run `make lint-crosstown`
**Then** golangci-lint v2.9.0 runs on crosstown packages
**And** no linting errors are found

**Given** linting runs in CI
**When** pull requests are submitted
**Then** linting failures block merging
**And** lint report is posted as PR comment

### Story 12.5: Implement comprehensive integration testing

As a **development team**,
I want **integration tests for all major crosstown features**,
So that **component interactions are validated**.

**Acceptance Criteria:**

**Given** integration tests exist with build tag
**When** I run `go test -tags=integration ./internal/crosstown/...`
**Then** integration tests execute:
- SPSP handshake with real connector
- BTP communication flow
- Relay subscription and events
- Payment channel lifecycle
- Job correlation persistence

**Given** integration tests use testcontainers
**When** tests run
**Then** required services are started automatically
**And** tests clean up containers after completion

**Given** integration tests validate NFRs
**When** reviewing test assertions
**Then** tests verify:
- Payment latency <500ms (NFR-P1)
- TOON extraction <1ms (NFR-P2)
- Channel opening <30s (NFR-P5)
- All security requirements

