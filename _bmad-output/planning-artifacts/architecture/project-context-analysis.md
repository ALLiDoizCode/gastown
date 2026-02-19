# Project Context Analysis

## Requirements Overview

**Functional Requirements (58 FRs across 8 categories):**

This integration adds crosstown network participation to gastown through 58 functional requirements spanning:

1. **Relay Communication (FR1-FR5)**: NIP-01 WebSocket subscription to crosstown relay, TOON event reception, automatic reconnection with exponential backoff, real-time connection status display

2. **Payment Channel Management (FR6-FR14)**: Wallet configuration with encrypted private keys, automatic SPSP handshake, payment channel opening on Base Sepolia testnet, git-backed channel state persistence, balance monitoring with configurable thresholds, on-chain state recovery

3. **Job Coordination (FR15-FR26)**: DVM job request construction (kind:5xxx), ILP PREPARE packet sending with TOON payloads, FULFILL/REJECT handling, git-backed job correlation (job_id → callback_skill), result routing (kind:6xxx), sequential job chain execution, crash-resistant correlation state, Polecat execution in dedicated worktrees

4. **TOON Event Processing (FR27-FR32)**: Zero-parse pattern (Go passes raw TOON bytes to Claude Code), LLM-native TOON interpretation, kind field extraction for routing, skill invocation with raw payloads for kind:5xxx and kind:6xxx events

5. **Configuration & Setup (FR33-FR38)**: Environment variable configuration, multiple config sources with priority (CLI flags > env vars > config file > defaults), mandatory private key encryption (AES-256-GCM), runtime passphrase prompting without storage

6. **Observability & Monitoring (FR39-FR45)**: Bubbletea dashboard with crosstown network panel, git audit trail (commit per ILP payment), structured payment logging (JSONL with rotation), CSV/JSON export, payment statistics, prominent network identification (testnet vs mainnet)

7. **CLI Operations (FR46-FR54)**: Interactive mode (`gt mayor attach`), wallet/channel/relay management commands, payment history export, multi-format output (text/JSON/CSV), non-interactive automation support, shell completion generation

8. **Skills Extensibility (FR55-FR58)**: Zero-configuration skill registration (drop file in `.claude/skills/`), automatic skill discovery, raw TOON payload handling, extensibility without Mayor core changes

**Non-Functional Requirements (25 NFRs across 5 categories):**

**Performance:**
- ILP payment flow: <500ms PREPARE→FULFILL (NFR-P1)
- TOON kind extraction: <1ms per event (NFR-P2)
- WebSocket throughput: ≥10 events/sec without drops (NFR-P3)
- Dashboard updates: <100ms latency for state changes (NFR-P4)
- Channel opening: <30s for SPSP handshake (NFR-P5)

**Security:**
- Private key encryption at rest: AES-256-GCM mandatory (NFR-S1) - CRITICAL
- Passphrase handling: Runtime prompt only, never logged/stored (NFR-S2)
- BTP connections: TLS (wss://) with certificate validation (NFR-S3)
- Channel balance limits: Max 100 M2M testnet (configurable) (NFR-S4)
- File permissions: 0600 for channel state files (NFR-S5)
- Tamper-evident audit: Git commit integrity (NFR-S6)
- Price validation: Warn if basePricePerByte >100 units/byte (NFR-S7)
- Network identification: Prominent testnet/mainnet display (NFR-S8)

**Reliability:**
- Crash recovery: Git state survives crashes, zero data loss (NFR-R1)
- Payment channel recovery: Reconstruct from on-chain if git lost (NFR-R2)
- WebSocket reconnection: Exponential backoff (1s→30s with jitter) (NFR-R3)
- ILP retry: Max 3 retries with backoff (NFR-R4)
- Job timeouts: Default 300s, fail gracefully (NFR-R5)
- Git serialization: No concurrent worktree operations (NFR-R6)
- Graceful degradation: Continue local-only if relay unavailable (NFR-R7)

**Integration:**
- Protocol compliance: NIP-01, BTP, ILPv4, TOON spec (NFR-I1-I4)
- Gastown integration: Cobra commands, Bubbletea TUI, git worktrees, skills system (NFR-I5-I8)
- EVM chain support: Base Sepolia testnet via Web3 RPC (NFR-I9)
- Config system extension: Seamless TOML/env var integration (NFR-I10)

**Scalability:**
- Concurrent agents: Max 20-30 Polecats (existing gastown limit) (NFR-SC1)
- Payment channels: ≥5 concurrent channels (NFR-SC2)
- Event subscriptions: ≥10 event kinds simultaneously (NFR-SC3)
- Job correlation: ≥1000 active job mappings without bloat (NFR-SC4)
- Log retention: 30-day rotation without manual intervention (NFR-SC5)

**Scale & Complexity:**

- **Primary domain**: CLI tool with daemon processes, WebSocket networking, blockchain integration
- **Complexity level**: High (fintech + brownfield + multi-protocol integration)
- **Estimated architectural components**: 8 major components
  1. NIP-01 WebSocket client (relay communication)
  2. BTP client (ILP packet transport)
  3. SPSP handshake handler (payment channel negotiation)
  4. Payment channel manager (state persistence + on-chain ops)
  5. TOON routing layer (zero-parse to Claude Code)
  6. Job correlation tracker (git-backed mappings)
  7. Crosstown CLI commands (`gt crosstown` group)
  8. Bubbletea dashboard extension (crosstown panel)

## Technical Constraints & Dependencies

**Brownfield Integration Constraints:**
- Must integrate with existing 682-file, 58-package Go codebase
- Cannot break existing commands or agent workflows
- Must follow existing patterns: Cobra CLI structure, Bubbletea MVU, git-backed hooks
- Git operations must remain serialized (existing worktree locking respected)
- Mayor's skills system accepts TOON bytes without API refactoring

**Language & Build Constraints:**
- Go 1.24.2 strict requirement
- Always use `make build` (never `go build` - version info required)
- Always use `make install` (never `go install` - removes stale binaries)
- E2E tests ONLY in Docker: `make test-e2e-container`
- golangci-lint v2.9.0 pinned (do not upgrade without testing)

**Protocol Compliance Requirements:**
- NIP-01 (Nostr Basic Protocol): REQ/EVENT/EOSE/CLOSE message handling
- BTP (Bilateral Transfer Protocol): Packet framing, WebSocket transport
- ILPv4 (RFC 0027): PREPARE/FULFILL/REJECT packet structure, OER encoding
- TOON format: Binary encoding following https://github.com/nicholasgasior/toon spec

**External System Dependencies:**
- Crosstown relay: WebSocket endpoint (ws://relay.crosstown.network)
- Crosstown connector: BTP endpoint (btp+ws://connector.crosstown.network:3000)
- Base Sepolia testnet: EVM chain ID 84532, RPC endpoint (https://sepolia.base.org)
- M2M token contract: 0x39eaF99Cd4965A28DFe8B1455DD42aB49D0836B9

**Gastown-Specific Constraints:**
- Max 20-30 concurrent Polecats (performance limit)
- Git worktrees consume full disk checkout each
- Bubbletea TUI: Immutable models, pure update functions (MVU pattern)
- No sudo required for normal operations

## Cross-Cutting Concerns Identified

**1. Security & Compliance (Fintech Domain)**
- Private key encryption at rest (AES-256-GCM) - CRITICAL
- TLS for all network transport (BTP, WebSocket)
- Tamper-evident audit trails (git commits for payments)
- Fraud prevention (balance limits, price validation, timeouts)
- Testnet-only MVP with mainnet compliance roadmap

**2. Reliability & Crash Recovery**
- Git-backed state for crash survival (payment channels, job correlation)
- Automatic reconnection (relay, connector) with exponential backoff
- On-chain state reconstruction if git lost
- Job timeout handling and sequential chain resumption
- Zero data loss guarantee

**3. Observability & Debugging**
- Bubbletea dashboard real-time updates (connection, balance, jobs)
- Git audit trail (commit per payment with metadata)
- Structured logging (payment_log.jsonl with rotation)
- Payment export (CSV/JSON for analysis)
- Network identification (testnet/mainnet prominence)

**4. Integration & Protocol Compliance**
- Must comply with 4 protocol specs (NIP-01, BTP, ILPv4, TOON)
- Must integrate with existing gastown (Cobra, Bubbletea, git worktrees, skills)
- Must support EVM chain operations (Base Sepolia)
- Configuration must extend existing system seamlessly

**5. Performance & Scalability**
- Sub-millisecond TOON kind extraction (high-throughput event processing)
- Sub-500ms ILP payments (non-blocking agent work)
- Concurrent Polecat limit (20-30) - existing constraint
- Efficient git operations (avoid worktree bloat)

**6. Developer Experience & Extensibility**
- Zero-configuration skill registration (drop file → automatic discovery)
- Shell completion for new commands (bash, zsh, fish, PowerShell)
- Multi-format output (text, JSON, CSV for automation)
- Clear error messages with actionable suggestions
