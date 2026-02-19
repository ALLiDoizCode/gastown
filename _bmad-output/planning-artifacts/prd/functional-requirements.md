# Functional Requirements

## 1. Relay Communication

**FR1:** Mayor can connect to crosstown relay via NIP-01 WebSocket protocol

**FR2:** Mayor can subscribe to specific Nostr event kinds with subscription filters (kind:5xxx DVM jobs, kind:6xxx DVM results)

**FR3:** Mayor can receive TOON-encoded events from relay via WebSocket

**FR4:** Mayor can reconnect to relay automatically on connection loss with exponential backoff retry logic

**FR5:** Mayor can display relay connection status in real-time

## 2. Payment Channel Management

**FR6:** Operator can configure wallet with encrypted private key via CLI commands and environment variables

**FR7:** Mayor can perform SPSP handshake with crosstown connector automatically on startup

**FR8:** Mayor can negotiate settlement chain (Base Sepolia testnet) during SPSP handshake

**FR9:** Mayor can open payment channel with M2M token deposit on negotiated settlement chain

**FR10:** Mayor can persist payment channel state (channel ID, balance, settlement address) in git worktree

**FR11:** Mayor can monitor payment channel balance and display warnings when balance drops below 10 M2M tokens (configurable threshold)

**FR12:** Mayor can close payment channel and withdraw remaining funds

**FR13:** Mayor can reconstruct payment channel state from on-chain events if git state is lost

**FR14:** Mayor can validate relay pricing (basePricePerByte) during SPSP handshake and warn if exceeds 100 satoshis/byte (configurable threshold)

## 3. Job Coordination

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

## 4. TOON Event Processing

**FR27:** Mayor can pass raw TOON byte arrays to Claude Code with zero parsing in Go networking layer

**FR28:** Claude Code can interpret TOON structure directly using LLM-native processing (pattern recognition on binary byte sequences without explicit schema definitions, leveraging TOON's structural regularity to extract fields with fewer tokens than JSON parsing)

**FR29:** Claude Code can extract event kind field from TOON payload for routing decisions

**FR30:** Claude Code can route raw TOON payloads to appropriate skills based on extracted kind

**FR31:** Skills can handle kind:5xxx DVM job requests with raw TOON payloads

**FR32:** Skills can handle kind:6xxx DVM result events with raw TOON payloads

## 5. Configuration & Setup

**FR33:** Operator can configure wallet settings via environment variables (address, encrypted private key, RPC URL)

**FR34:** Operator can configure crosstown settings (relay URL, connector endpoint) via environment variables, config file (~/.gt/crosstown.toml), or CLI commands

**FR35:** Mayor can enforce private key encryption at rest and refuse to start if keys unencrypted unless override flag explicitly set

**FR36:** Mayor can prompt for encryption passphrase at runtime without storing or logging passphrase

**FR37:** Mayor can apply configuration from multiple sources with defined priority (CLI flags > env vars > config file > defaults)

**FR38:** Operator can override security defaults with explicit warning flags (GASTOWN_ALLOW_UNENCRYPTED_KEYS)

## 6. Observability & Monitoring

**FR39:** Mayor can display Bubbletea dashboard with crosstown network panel showing connection status, channel balance, subscribed events, and recent activity

**Dashboard Layout:**
```
╔═══════════════════════════════════════════════════════════════════════════╗
║ Mayor Dashboard - gastown v0.7.0                    Base Sepolia Testnet ║
╠═══════════════════════════════════════════════════════════════════════════╣
║                                                                           ║
║ ┌─ Crosstown Network ─────────────────────────────────────────────────┐  ║
║ │                                                                      │  ║
║ │ Status: ✅ Connected                                                 │  ║
║ │ Relay:  ws://relay.crosstown.network                                │  ║
║ │                                                                      │  ║
║ │ Payment Channel                                                      │  ║
║ │ ├─ Balance:    49.99 M2M tokens                                      │  ║
║ │ ├─ Chain:      Base Sepolia (84532)                                 │  ║
║ │ ├─ Channel ID: 0x7f3a2b...                                          │  ║
║ │ └─ Status:     Open                                                  │  ║
║ │                                                                      │  ║
║ │ Subscriptions                                                        │  ║
║ │ ├─ kind:5xxx (DVM Job Requests)      ✓ Active                       │  ║
║ │ └─ kind:6xxx (DVM Results)           ✓ Active                       │  ║
║ │                                                                      │  ║
║ │ Recent Activity                                                      │  ║
║ │ ├─ 14:23:45 📤 Posted job-5abc123 (875 bytes, 0.00875 M2M)          │  ║
║ │ ├─ 14:24:32 📥 Received result from Mayor-Beta                      │  ║
║ │ └─ 14:24:33 ✅ Job completed successfully                           │  ║
║ │                                                                      │  ║
║ └──────────────────────────────────────────────────────────────────────┘  ║
║                                                                           ║
║ [q] Quit  [r] Refresh  [j] Job History  [c] Channel Details  [h] Help   ║
╚═══════════════════════════════════════════════════════════════════════════╝
```

**FR40:** Mayor can log every ILP payment to git audit trail with commit per payment including job ID, amount, bytes, and status

**FR41:** Mayor can log structured payment data to append-only log file (payment_log.jsonl) with rotation

**FR42:** Mayor can export payment history to CSV and JSON formats with date filtering

**FR43:** Mayor can display payment statistics (total payments, total M2M spent, average cost per event)

**FR44:** Mayor can log payment channel state changes (open/close events) with on-chain transaction hashes

**FR45:** Mayor can display active network (Base Sepolia testnet vs mainnet) prominently to prevent deployment confusion

## 7. CLI Operations

**FR46:** Operator can run Mayor in interactive mode (gt mayor attach) with real-time Bubbletea dashboard

**FR47:** Operator can manage wallet via CLI commands (configure, check balance, export encrypted key)

**FR48:** Operator can manage payment channels via CLI commands (open, check status, close, list all channels)

**FR49:** Operator can manage relay connection via CLI commands (subscribe, disconnect, check status)

**FR50:** Operator can export payment history via CLI command (gt mayor export-payments) with format and date options

**FR51:** All CLI commands can output in multiple formats (human-readable text, JSON, CSV) via --format flag

**FR52:** All CLI commands can run in non-interactive mode with --yes flag to skip confirmations for automation

**FR53:** Operator can generate shell completion scripts for bash, zsh, fish, and PowerShell

**FR54:** Operator can automate crosstown operations via shell scripts using environment variable configuration and non-interactive flags

## 8. Skills Extensibility

**FR55:** Developer can create skill files in .claude/skills/ directory with zero configuration

**FR56:** Claude Code can automatically discover skills from .claude/skills/ and route events based on skill-declared supported kinds

**FR57:** Skills can receive raw TOON payloads for processing without requiring JSON conversion

**FR58:** Developer can extend Mayor to handle new Nostr event kinds by adding skill files with zero changes to Mayor core code

