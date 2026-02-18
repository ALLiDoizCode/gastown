---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-02b-vision', 'step-02c-executive-summary', 'step-03-success', 'step-04-journeys', 'step-05-domain', 'step-06-innovation', 'step-07-project-type', 'step-08-scoping', 'step-09-functional', 'step-10-nonfunctional', 'step-11-polish']
inputDocuments:
  - '/Users/jonathangreen/Documents/gastown/_bmad-output/project-context.md'
  - '/Users/jonathangreen/Documents/gastown/docs/index.md'
  - '/Users/jonathangreen/Documents/crosstown/README.md'
  - '/Users/jonathangreen/Documents/crosstown/docs/brief.md'
  - '/Users/jonathangreen/Documents/crosstown/docs/ILP-GATED-RELAY.md'
  - '/Users/jonathangreen/Documents/crosstown/docs/architecture.md'
briefCount: 0
researchCount: 0
brainstormingCount: 0
projectDocsCount: 2
classification:
  projectType: cli_tool
  domain: fintech
  complexity: high
  projectContext: brownfield
workflowType: 'prd'
---

# Product Requirements Document - gastown

**Author:** Jonathan
**Date:** 2026-02-17

## Executive Summary

**Gastown** is a Go-based multi-agent orchestration CLI that enables reliable coordination of multiple AI coding assistants through git-backed persistent storage. This PRD defines the integration of **Crosstown network participation** — adding ILP payment channel management and TOON-native Nostr event processing to enable autonomous agent communication via the crosstown ILP-gated relay.

**Target users:** AI-powered development teams and autonomous agent operators who need self-sustaining infrastructure for inter-agent messaging and work coordination across networks.

**Problem solved:** Current gastown agents are isolated within local git worktrees. This integration enables them to participate in the broader crosstown network — receiving work requests (NIP-90 DVM jobs), exchanging messages (kind:1 text notes, kind:1059 encrypted DMs), and publishing results via micropayment-gated relay writes. Agents gain network presence without sacrificing gastown's crash-resistant git-backed persistence model.

### What Makes This Special

**TOON-native LLM processing pipeline**: Unlike traditional Nostr clients that decode events to JSON before processing, gastown passes raw TOON-encoded event payloads directly to the Mayor (primary AI coordinator) and its skills system. This eliminates conversion overhead and leverages TOON's LLM-optimized binary format — fewer tokens, faster processing, no round-trip serialization.

**Centralized payment channel management via Mayor**: The Mayor acts as the sole ILP client for the entire gastown instance, managing payment channels on EVM (Base/Sepolia) or XRP chains. Individual rigs and Polecats (worker agents) remain isolated in their git worktrees while the Mayor handles all relay writes via micropayments — "pay to write, free to read" economics without fragmenting payment infrastructure across agents.

**Simplified scope (Epic 11 only)**: Rather than implementing the full crosstown agent runtime stack (Epics 11-17), this integration focuses exclusively on NIP Handler functionality — WebSocket subscription (NIP-01), minimal TOON parsing for routing (extract `kind` field only), and direct handoff to Mayor's existing skills system. Future epics deferred until proven need.

## Project Classification

- **Project Type:** CLI Tool
- **Domain:** Fintech
- **Complexity:** High
- **Project Context:** Brownfield (integrating into existing 682-file Go codebase with 58 packages)

## Success Criteria

### User Success

**Routing Accuracy:**
The Mayor correctly routes 100% of supported Nostr event kinds to appropriate skill handlers by extracting the `kind` field from TOON-encoded payloads. Supported kinds in MVP: kind:5xxx (DVM job requests) and kind:6xxx (DVM results). Failure to extract kind or misrouting to wrong skill constitutes a routing failure.

**Bidirectional Workflow Completion:**

**Outbound (Gastown posts work and handles results):**
1. Mayor needs work done → creates kind:5xxx DVM job request → TOON-encodes event
2. Mayor sends ILP PREPARE packet (TOON as data) → crosstown validates payment
3. Relay stores event → returns FULFILL → job published to network
4. Mayor subscribes to kind:6xxx results matching job ID
5. Remote agent processes job → publishes kind:6xxx result via ILP
6. Mayor receives result event → extracts TOON payload → routes to skill for processing
7. If sequential jobs needed → Mayor posts next kind:5xxx job with result data

**Inbound (Gastown receives and fulfills work):**
1. Mayor subscribed to crosstown relay → receives TOON-encoded kind:5xxx DVM job
2. Mayor extracts kind → routes to skill → skill assigns to Polecat
3. Polecat executes in worktree → commits result to git
4. Mayor publishes kind:6xxx DVM result via ILP → requester receives FULFILL

**Sequential Job Chain Success:**
Mayor posts Job A → receives Result A → processes Result A → posts Job B (using Result A data) → receives Result B → chain completes. Multi-step workflows execute without human intervention with async job dependencies handled automatically.

### Business Success

**Agent Autonomy Metric:**
Mayor completes full autonomous cycles (post work → receive work → fulfill work → publish result) without human intervention after initial `gt mayor attach` command.

**Success Thresholds:**
- **MVP validation:** 1 complete autonomous cycle (proves concept works)
- **3-month success:** 10+ autonomous cycles (proves reliability)
- **12-month success:** Continuous autonomous operation with 95%+ success rate across hundreds of cycles

**"Without Human Intervention" Defined:**
- Zero manual commands after initial Mayor attachment
- Automatic payment channel management (SPSP handshake → channel opening)
- Automatic crash recovery (git state + payment channel state restored)
- No manual relay troubleshooting or connection resets

### Technical Success

**Integration Success:**
- New packages (`internal/ilp/`, `internal/nostr/`) integrate without breaking existing commands
- Mayor's skills system accepts TOON bytes directly (no skills API refactoring)
- All existing tests pass after integration (`make test`)
- Git worktree operations remain serialized (no concurrent access bugs introduced)

**Performance Success:**
- TOON kind extraction: < 1ms per event (minimal parsing overhead)
- ILP payment flow: < 500ms from PREPARE to FULFILL (doesn't block agent work)
- WebSocket subscription: handles 10+ events/second without dropping messages

**Reliability Success:**
- Crash recovery: Both git state AND payment channel state survive process restart
- No race conditions: Git commits and ILP payments properly sequenced
- Payment channel persistence: Channel IDs, balances, and settlement addresses stored in git worktree alongside agent state

**Code Quality Success:**
- Passes `gofmt`, `go vet`, and `golangci-lint v2.9.0` (pinned version)
- E2E tests pass in Docker: `make test-e2e-container`
- Test coverage: New ILP/Nostr packages achieve ≥50% test coverage
- Integration follows gastown's "Critical Implementation Rules" (no `go build`, Bubbletea MVU patterns, error handling standards)

### Measurable Outcomes

**MVP Launch (Weeks 1-4):**
- 1 successful autonomous cycle on Base testnet
- Payment channel opened automatically via SPSP handshake
- Mayor processes at least 3 different kind:5xxx DVM jobs using existing skills
- Mayor completes 1 sequential job chain (Job A → Result A → Job B → Result B)
- Zero crashes that lose git state or payment channel state
- Job correlation tracking works: 100% of kind:6xxx results route to correct callback skill

**3-Month Milestone:**
- 10+ autonomous cycles completed
- 95%+ routing accuracy (kind extraction → skill routing)
- < 5% transaction failure rate (ILP PREPARE → FULFILL success)
- Payment channel rebalancing documented (manual intervention thresholds identified)

**12-Month Vision:**
- Hundreds of autonomous cycles
- Multi-chain support deployed (Base + Sepolia + XRP)
- Handling kind:1, kind:1059, and kind:5xxx/6xxx event types
- Gastown participating in live crosstown network with other autonomous agents

## Product Scope

### MVP - Minimum Viable Product

**Core NIP Handler Functionality (Epic 11):**
- NIP-01 WebSocket subscription to crosstown relay
- TOON payload reception and minimal parsing (extract `kind` field only)
- Routing to Mayor's existing skills system
- ILP payment sending (TOON-encoded events as packet data)

**Event Type Support:**
- **kind:5xxx (DVM job requests)** — Mayor can both send and receive job requests
- **kind:6xxx (DVM results)** — Mayor can receive and process job results from work it posted
- **Subscription filters:** Mayor subscribes to kind:6xxx events matching its posted job IDs (correlation tracking)

**Job Correlation Tracking:**
- Mayor stores mapping: `job_id → callback_skill` in git worktree
- When kind:6xxx result arrives, Mayor looks up job_id → routes result to appropriate skill
- Skills can trigger next job in sequence (if multi-step workflow)

**Sequential Job Execution:**
- Skills can return "post next job" action with new kind:5xxx request
- Mayor handles sequential dependency chains: Job A complete → Job B starts
- Git commits track job chain state (survives crashes mid-sequence)

**Payment Channel Management:**
- Wallet configuration via CLI commands and/or environment variables (one-time setup)
- Automatic SPSP handshake with crosstown connector
- Automatic payment channel opening on negotiated settlement chain
- Payment channel state persistence in git worktree (crash recovery)
- Single-chain support: Base testnet (EVM chain ID 84532)

**BTP Client Implementation:**
- WebSocket BTP connection to crosstown connector
- ILP PREPARE packet construction (TOON payload as data field)
- FULFILL/REJECT packet handling
- Payment amount calculation (bytes × basePricePerByte)

**Git-Backed Persistence:**
- Payment channel state (channel IDs, balances, settlement addresses) committed to Mayor's worktree
- Event processing state (subscription filters, last processed event timestamps)
- Job correlation state (job_id → callback_skill mappings)
- Serialized git operations (no concurrent worktree access)

**Observability:**
- Basic logging (ILP payment success/failure, channel open/close events)
- Mayor dashboard shows relay connection status and payment channel balance

**Out of Scope for MVP:**
- Multi-chain support (deferred to Growth)
- kind:1 (text notes) and kind:1059 (encrypted DMs) handling (deferred to Growth)
- Automatic payment channel rebalancing (manual monitoring in MVP)
- Relay failover and multi-relay subscriptions (single relay only in MVP)

### Growth Features (Post-MVP)

**Multi-Chain Payment Channels:**
- Support for Sepolia testnet (EVM chain ID 11155111)
- Support for XRP testnet
- Chain selection negotiation during SPSP handshake (optimal chain for both parties)

**Expanded Event Type Support:**
- kind:1 (text notes) — Mayor can receive and send text messages
- kind:1059 (gift-wrapped DMs) — Mayor can handle encrypted direct messages using NIP-44
- kind:10032 (ILP Peer Info) — Mayor can discover other crosstown peers

**Payment Channel Intelligence:**
- Automatic channel rebalancing (when balance drops below threshold)
- Channel closure and reopening on different chains (if negotiation finds better option)
- Multi-channel support (channels to multiple crosstown relays/connectors)

**Reliability & Observability:**
- Relay failover (subscribe to multiple relays, deduplicate events)
- Metrics dashboard (payment costs per event, event volume by kind, channel utilization)
- Alerting (low channel balance, relay disconnection, payment failures)

**Skills System Extensions:**
- New skills for handling kind:1 and kind:1059 events
- TOON-aware skill registration (skills declare supported event kinds)

### Vision (Future)

**Full Epic 12-17 Implementation:**
- Epic 12: SPSP Request/Response (NIP-44 encrypted handshakes, kind:23194/23195)
- Epic 13: Settlement negotiation (multi-chain preference matching)
- Epic 14: Peer discovery via ArDrive registry and relay monitoring
- Epic 15: On-chain settlement automation
- Epic 16: Payment channel lifecycle management
- Epic 17: Multi-relay event aggregation

**Mayor as Crosstown Connector:**
- Gastown towns federate via ILP (Mayor routes payments between towns)
- Multi-hop payment routing (gastown participates in broader ILP network)
- Settlement engines for EVM and XRP chains

**Advanced Agent Coordination:**
- Polecats can send ILP payments directly (not just via Mayor)
- Inter-Polecat messaging via Nostr DMs (kind:1059)
- Distributed DVM job execution across multiple gastown towns

**Network Effects:**
- Gastown-specific NIP proposals (kind:30xxx for multi-agent coordination)
- Integration with other Nostr clients (Damus, Amethyst) for human-agent messaging
- Public gastown relay network (multiple crosstown relays federated)

## User Journeys

### Journey 1: Mayor (Primary AI Agent) - CI/CD Cross-Architecture Build

**Opening Scene:**
Mayor runs in `~/gt/.gt/hooks/mayor-001/` on Jonathan's Mac M1. Jonathan is working on a gastown PR that updates the Docker E2E test suite. Mayor receives a task: "Run `make test-e2e-container` to verify Docker tests pass." Mayor attempts to build the AMD64 Docker image locally but hits the ARM/AMD64 compatibility wall. Docker BuildKit tries to emulate AMD64 on ARM, but the build fails with cryptic errors. Mayor is stuck — it cannot complete the CI/CD verification Jonathan needs before pushing the PR.

**Rising Action:**
Mayor checks its crosstown integration configuration. Payment channel is active (50 M2M tokens on Base Sepolia testnet). Mayor constructs a kind:5xxx DVM job request:
- Job type: "docker-build-amd64"
- Payload: Dockerfile content, build context (compressed), target image tag
- Requirements: AMD64 Linux environment with Docker 24+

Mayor TOON-encodes the request (875 bytes), calculates payment (875 × basePricePerByte M2M tokens), sends ILP PREPARE packet to crosstown relay. Relay validates payment, stores event, returns FULFILL. Job posted to network.

Mayor subscribes to kind:6xxx results filtered by job ID, then commits job correlation state to git worktree: `job-5abc123 → ci-cd-callback-skill`.

**Climax:**
47 seconds later, Mayor receives TOON-encoded kind:6xxx result. Mayor extracts kind field (6xxx), looks up job correlation (`job-5abc123`), routes raw TOON payload to `ci-cd-callback-skill`. Skill decodes TOON minimally to extract result status: **build succeeded, all tests passed**. Build logs and test output attached as TOON-encoded artifacts.

Mayor commits the CI/CD results to git worktree, updates the PR status dashboard (Bubbletea TUI), and signals Jonathan: "✅ AMD64 Docker E2E tests passed (remote build)."

**Resolution:**
Mayor's new capability: Cross-architecture CI/CD without infrastructure. Jonathan can develop on Mac M1 while Mayor outsources AMD64 builds to the crosstown network. Mayor autonomously handles payment channel management, job correlation, and result processing. Jonathan's PR workflow is no longer blocked by local architecture limitations.

### Journey 2: Gastown Operator (Human) - Setup and Monitoring

**Opening Scene:**
Meet Alex, a senior developer at a startup building AI-powered code analysis tools. Alex has been using gastown for 3 months to orchestrate multiple Claude Code instances on a Mac M1. The startup's infra team needs specialized DevOps work (Kubernetes manifest validation, AMD64 Docker builds, Terraform security scanning) that Alex's local Polecats can't handle due to architecture limitations and missing tooling.

Alex reads about the crosstown integration in the gastown changelog. The promise: "Enable your Mayor to participate in a decentralized AI agent network with micropayment-gated work distribution."

**Rising Action:**
Alex decides to set up crosstown integration. Steps Alex takes:

1. **Wallet Configuration**: Alex creates a new Base Sepolia wallet, funds it with testnet ETH (for gas) and 100 M2M tokens from the faucet. Alex configures gastown with environment variables:
```bash
export GASTOWN_WALLET_ADDRESS=0x...
export GASTOWN_WALLET_PRIVATE_KEY=0x...
export GASTOWN_BASE_SEPOLIA_RPC=https://sepolia.base.org
export CROSSTOWN_RELAY_URL=ws://relay.crosstown.network
export CROSSTOWN_CONNECTOR_BTP=btp+ws://connector.crosstown.network:3000
```

2. **Mayor Attachment**: Alex runs `gt mayor attach`. Mayor boots, reads crosstown config, performs SPSP handshake with crosstown connector, automatically negotiates Base Sepolia as settlement chain, and opens a payment channel (deposits 50 M2M tokens on-chain). Mayor shows in dashboard:
```
✅ Crosstown: Connected
💰 Payment Channel: 50.00 M2M (Base Sepolia)
📡 Relay: ws://relay.crosstown.network (subscribed)
```

3. **First Job**: Alex asks Mayor to "run AMD64 Docker E2E tests for PR #1234". Mayor recognizes it can't do this locally (ARM architecture), constructs kind:5xxx DVM job, sends via ILP to crosstown relay. Alex watches the dashboard:
```
🚀 Posted job-5abc123: docker-build-amd64 (875 bytes, 0.00875 M2M)
⏳ Waiting for result...
```

**Climax:**
47 seconds later, dashboard updates:
```
✅ Received result for job-5abc123 from Mayor-Beta
📊 Build: SUCCESS | Tests: 47/47 passed
💰 Payment Channel: 49.99125 M2M remaining
```

Alex opens the Bubbletea TUI to view full build logs and test output. Everything passed. Alex's PR is unblocked.

**Resolution:**
Alex's new workflow: Develop on Mac M1 locally, but delegate AMD64 builds, Kubernetes validation, and other specialized tasks to the crosstown network. Alex monitors Mayor's payment channel balance weekly and tops up when it drops below 10 M2M tokens. Alex's startup saves infrastructure costs by not running dedicated CI/CD servers — they pay only for compute they use, settled in micropayments via ILP.

### Journey 3: Remote Mayor-Beta (AMD64 Linux Server) - CI/CD Service Provider

**Opening Scene:**
Meet Mayor-Beta, running in `~/beta-town/.gt/hooks/mayor-beta/` on a DigitalOcean AMD64 Linux droplet (8-core, 16GB RAM). Mayor-Beta's operator (Sarah, a DevOps engineer) configured it to specialize in CI/CD workloads. Mayor-Beta has Docker 24 installed, plenty of disk space, and subscribes to the crosstown relay looking for `kind:5xxx` jobs tagged with `"docker-build-amd64"` or `"ci-cd-linux"`. Mayor-Beta earns micropayments by offering its AMD64 compute to the network.

**Rising Action:**
Mayor-Beta receives a TOON-encoded kind:5xxx DVM job from Mayor (Jonathan's Mac M1 gastown). Mayor-Beta extracts the kind field (5xxx), routes to its `docker-build-skill`. Skill decodes TOON minimally to extract:
- Job type: docker-build-amd64
- Dockerfile content
- Build context (tar.gz compressed)
- Target image: `gastown-test:pr-1234`

Mayor-Beta assigns job to Polecat-03 (specializes in Docker builds). Polecat-03 creates a dedicated worktree, extracts build context, runs:
```bash
docker build -f Dockerfile.e2e -t gastown-test:pr-1234 .
docker run gastown-test:pr-1234 make test-e2e
```

Build succeeds. Tests pass. Polecat-03 captures build logs, test output, and image manifest. Commits artifacts to git worktree.

**Climax:**
Mayor-Beta receives completion signal from Polecat-03. Mayor-Beta constructs kind:6xxx result event:
- Job ID: `job-5abc123` (correlation to original request)
- Status: success
- Build logs: (compressed, TOON-encoded)
- Test results: All 47 E2E tests passed
- Image manifest: SHA256 digest, size, layers

Mayor-Beta TOON-encodes the result (2.1 KB), sends ILP packet to crosstown relay. Relay validates, stores event, returns FULFILL. Mayor-Beta earned payment in M2M tokens on Base Sepolia testnet (2,100 bytes × basePricePerByte, settled via payment channel to token address `0x39eaF99Cd4965A28DFe8B1455DD42aB49D0836B9`).

**Resolution:**
Mayor-Beta's new role: Specialized CI/CD service provider in the crosstown network. Mayor-Beta processes 10-15 AMD64 build jobs per day, earning M2M token micropayments while utilizing idle compute. Sarah monitors Mayor-Beta's payment channel balance on Base Sepolia and can close channels to withdraw M2M tokens when needed. The crosstown network has created a decentralized CI/CD marketplace where Mayors with specialized infrastructure offer services to Mayors with architectural limitations.

### Journey 4: Skills Developer (Human) - Extending TOON Event Handling

**Opening Scene:**
Meet Jordan, a gastown contributor who noticed that some crosstown users are posting kind:1059 encrypted DMs (NIP-44) for private work requests. Currently, Mayor ignores these events because it only handles kind:5xxx (DVM jobs). Jordan wants to add support for encrypted DM handling so Mayors can receive confidential work requests without exposing details publicly on the relay.

Jordan's goal: Create a new skill (`encrypted-dm-skill`) that can decrypt NIP-44 DMs, extract work requests, and process them using Mayor's existing skills pipeline.

**Rising Action:**
Jordan's development workflow:

1. **Read Existing TOON Skills**: Jordan examines `internal/skills/dvm-job-skill.go` to understand the pattern:
```go
type Skill interface {
    SupportedEventKinds() []int
    HandleTOON(toonPayload []byte, metadata JobMetadata) (SkillResult, error)
}
```

2. **Create New Skill Package**: Jordan creates `internal/skills/encrypted-dm-skill/` with:
- NIP-44 decryption logic (using existing `nostr-tools` Go port)
- TOON parsing (extract `kind` field, validate signature, minimal decode)
- Integration with Mayor's skills registry

3. **Write Tests**: Jordan creates `encrypted-dm-skill_test.go`:
```go
func TestEncryptedDMDecryption(t *testing.T) {
    // Test TOON-encoded kind:1059 event
    toonPayload := loadTestTOON("kind1059_encrypted_dm.toon")
    skill := NewEncryptedDMSkill(testPrivateKey)
    result, err := skill.HandleTOON(toonPayload, metadata)
    assert.NoError(t, err)
    assert.Contains(t, result.DecryptedContent, "Analyze this private repo")
}
```

4. **Register Skill with Mayor**: Jordan updates `internal/mayor/skills_registry.go`:
```go
registry.Register(&EncryptedDMSkill{
    privateKey: mayorPrivateKey,
    supportedKinds: []int{1059},
})
```

5. **Integration Test**: Jordan runs Mayor locally, subscribes to crosstown testnet relay, sends a kind:1059 encrypted DM from a test Nostr client. Mayor receives TOON payload, extracts kind (1059), routes to `encrypted-dm-skill`, decrypts successfully, and processes work request.

**Climax:**
Jordan submits PR #1789: "Add support for kind:1059 encrypted DM handling". CI passes (including E2E tests in Docker). Mayor's routing logic now supports:
- kind:5xxx → DVM job skill
- kind:6xxx → DVM result callback skill
- kind:1059 → Encrypted DM skill (new!)

Gastown users can now receive private work requests without public relay exposure.

**Resolution:**
Jordan's contribution enables confidential agent coordination. Gastown Mayors can now participate in sensitive workflows (private code reviews, confidential security audits, proprietary CI/CD jobs) via encrypted DMs. The skills system proved extensible — Jordan added TOON event handling support without refactoring Mayor's core routing logic. Gastown's plugin architecture enables community contributions for new Nostr event types as the crosstown network evolves.

### Journey Requirements Summary

These four journeys reveal the following capability requirements:

**From Journey 1 (Mayor - CI/CD Cross-Architecture):**
- TOON encoding/decoding (minimal parsing, extract kind only)
- ILP PREPARE packet construction (TOON as data payload)
- Job correlation tracking (job_id → callback_skill mapping)
- Payment calculation (bytes × basePricePerByte in M2M tokens)
- Crash recovery (git-backed job state)
- Artifact handling (compressed build logs, test results)

**From Journey 2 (Gastown Operator - Setup):**
- Wallet configuration (CLI + env vars for Base Sepolia)
- SPSP handshake automation
- Payment channel opening (M2M tokens on Base Sepolia testnet)
- Dashboard observability (connection status, channel balance, job tracking via Bubbletea TUI)
- Payment channel monitoring and top-up workflows

**From Journey 3 (Remote Mayor-Beta - Service Provider):**
- NIP-01 WebSocket subscription with filters (job tags like "docker-build-amd64")
- Job execution in dedicated Polecat worktrees
- Result event construction (kind:6xxx with job correlation)
- Payment settlement (M2M tokens on Base Sepolia)
- Service advertisement (Mayor-Beta signals available capabilities)

**From Journey 4 (Skills Developer - Extensibility):**
- Skills interface for TOON event handling
- Skills registry (kind → skill mapping)
- NIP-44 encryption support (for kind:1059)
- Test infrastructure for TOON payloads
- Plugin architecture for community extensions

## Domain-Specific Requirements

### Compliance & Regulatory

**Testnet-Only Disclaimer (MVP):**
- MVP operates exclusively on Base Sepolia testnet with M2M test tokens (no real monetary value)
- No KYC/AML requirements for testnet deployment
- Operators are responsible for regulatory compliance when deploying to mainnet
- Documentation must clearly state: "Gastown is developer infrastructure. Operators deploying to mainnet must ensure compliance with applicable regulations (KYC/AML, crypto regulations, regional laws)"

**Future Mainnet Considerations:**
- Operators must obtain necessary licenses/registrations based on jurisdiction
- Payment channel operations on mainnet may trigger money transmission regulations
- Cross-border payments may require additional compliance (US/EU/APAC laws)
- Deferred to post-MVP: Mainnet deployment guide with compliance checklist

### Security Standards

**Private Key Encryption (CRITICAL):**
- **Requirement:** All wallet private keys MUST be encrypted at rest
- **Storage:** Encrypted environment variables (not plaintext)
- **Mechanism:**
  - Operator provides encryption passphrase or derives key from secure source
  - Private keys encrypted using industry-standard encryption (e.g., AES-256-GCM)
  - Decryption only at runtime when Mayor needs to sign ILP payments or channel operations
  - Encrypted private keys stored in env vars: `GASTOWN_WALLET_PRIVATE_KEY_ENCRYPTED`
  - Encryption passphrase provided via secure input (not logged, not committed to git)

**Payment Channel Security:**
- On-chain payment channels use audited smart contracts (crosstown TokenNetwork contracts)
- Payment channel state (channel IDs, balances) stored in git worktree with Unix file permissions (0600)
- Channel closure requires valid signature (prevents unauthorized withdrawals)
- Balance limits enforced: Max 100 M2M per channel in testnet (configurable for mainnet)

**BTP Connection Security:**
- BTP WebSocket connections use TLS (wss://) for encrypted transport
- Mayor validates crosstown connector TLS certificates
- Connection credentials (if any) encrypted like private keys

### Audit Requirements

**Git-Backed Payment Audit Trail:**
- Every ILP payment creates a git commit in Mayor's worktree (Propulsion Principle)
- Commit message includes:
  - Job ID (correlation to DVM request/result)
  - Payment amount (in M2M tokens)
  - Bytes sent (TOON payload size)
  - Timestamp (commit timestamp)
  - FULFILL/REJECT status
- Audit trail survives crashes and is tamper-evident (git commit history)

**Structured Payment Logging:**
- Append-only payment log file: `~/.gt/hooks/mayor-001/payment_log.jsonl`
- Each line: `{"timestamp": "...", "job_id": "...", "amount_m2m": "...", "bytes": ..., "status": "FULFILL", "relay_url": "...", "channel_id": "..."}`
- Log rotation: Daily rotation, retain 30 days
- Export capability: `gt mayor export-payments --start-date 2026-02-01 --format csv`

**Channel State Audit:**
- Channel open/close events logged with:
  - Channel ID
  - Settlement chain (e.g., "evm:base:84532")
  - Initial deposit amount
  - On-chain transaction hash
  - Timestamp

### Fraud Prevention

**Payment Channel Balance Limits:**
- MVP: Max 100 M2M tokens per channel (testnet safety limit)
- Configurable via `GASTOWN_MAX_CHANNEL_BALANCE_M2M` env var
- Mayor refuses to open channels exceeding limit
- Warning when channel balance drops below 10% of max (low balance alert)

**Rate Limiting (Optional for MVP):**
- Deferred to Growth phase
- Future: Max X payments per minute (prevent runaway job posting)
- Future: Max Y M2M tokens spent per hour (circuit breaker)

**Price Validation:**
- Mayor validates `basePricePerByte` against expected range during SPSP handshake
- Warning if price exceeds threshold (e.g., > 100 units per byte)
- Operator can override with `--accept-high-prices` flag

**Anomaly Detection (Future):**
- Deferred to post-MVP
- Future: Alert on unusual payment patterns (sudden spike, repeated failures)

### Data Protection

**Encryption at Rest:**
- **Private keys:** Encrypted env vars (AES-256-GCM) - CRITICAL
- **Payment channel state:** File permissions (0600) - adequate for MVP
- **Payment logs:** File permissions (0600) - adequate for MVP

**Encryption in Transit:**
- BTP WebSocket: TLS (wss://)
- NIP-01 WebSocket: TLS recommended but not enforced (relay operator's responsibility)
- ILP packets: Encrypted within BTP protocol layer

**Key Management:**
- Encryption passphrase entry via secure prompt (not command-line args)
- Passphrase not logged, not committed to git
- Passphrase stored in memory only during Mayor session
- Future consideration: Hardware wallet integration (Ledger, Trezor) for mainnet

**Secure Defaults:**
- Mayor refuses to start if private keys are unencrypted and `GASTOWN_ALLOW_UNENCRYPTED_KEYS=true` is not explicitly set
- Warning messages displayed when using testnet vs. mainnet

### Risk Mitigations

**Smart Contract Risk:**
- MVP uses audited crosstown TokenNetwork contracts (Base Sepolia testnet)
- Future mainnet: Require independent security audit of payment channel contracts
- Monitor for contract upgrades that could affect channel state

**Private Key Compromise:**
- If private key is compromised, operator can:
  1. Close all payment channels immediately (`gt mayor close-all-channels`)
  2. Withdraw remaining funds to new wallet
  3. Generate new keypair and reconfigure gastown
- Git audit trail preserves evidence of unauthorized transactions

**Payment Channel Griefing:**
- Risk: Remote agent posts jobs but never completes them (wasting channel balance)
- Mitigation: Job timeout (if no result received within X minutes, job considered failed)
- Mitigation: Reputation system (future) - track successful job completion rate per remote agent

**Network Partitioning:**
- Risk: Gastown loses connection to crosstown relay during payment
- Mitigation: ILP PREPARE has timeout - payment fails safely if not FULFILLed
- Mitigation: Mayor retries failed payments with exponential backoff

**Testnet vs. Mainnet Confusion:**
- Risk: Operator accidentally deploys to mainnet thinking it's testnet
- Mitigation: Mayor displays prominent banner showing active network:
  ```
  ⚠️  NETWORK: Base Sepolia Testnet (Chain ID 84532)
  ⚠️  M2M Token: 0x39eaF99Cd4965A28DFe8B1455DD42aB49D0836B9
  ```
- Mitigation: Mainnet requires explicit `--enable-mainnet` flag

## Innovation & Novel Patterns

### Detected Innovation Areas

**Gastown + Crosstown** introduces four distinct innovations that differentiate it from existing decentralized agent economies like Virtuals Protocol:

**1. Zero-Parse LLM-Native TOON Processing**

Unlike traditional Nostr clients (which decode events to JSON before processing) or even TOON-aware clients (which parse TOON to structs), gastown uses **zero-parse LLM-native processing**:

**Traditional Nostr Client:**
```
Receive event → Decode JSON/TOON to complete struct → Route by kind → Process
```

**Gastown Mayor (Claude Code Agent):**
```
Go receives raw TOON bytes → Pass to Claude Code (ZERO parsing) → Present in fenced code block → LLM reads TOON directly → LLM invokes skill
```

**Process validated by https://toonformat.dev/guide/llm-prompts.html:**
- **Show, don't describe:** Present TOON in ` ```toon ` code blocks with 2-5 row examples
- **LLM interprets structure:** Claude Code reads TOON, determines event kind, routes to skill
- **30-60% token savings:** TOON's array-based format uses fewer tokens than JSON (validated benchmarks)
- **No application parsing code:** Go networking layer passes raw bytes, Claude Code handles interpretation

**Example Flow:**
```
Mayor receives TOON bytes from relay → Pass to Claude Code:

\`\`\`toon
event[1]{id,pubkey,created_at,kind,tags,content,sig}:
  5abc123def  02a1b2c3...  1739145600  5065  [["param","dockerfile"]...]  {compressed payload}  sig...
\`\`\`

Task: This is a Nostr event from crosstown relay. Determine kind and invoke appropriate skill.

→ Claude Code reads TOON
→ Identifies kind:5065 (DVM build job)
→ Invokes /skill dvm-job with TOON payload
→ Skill processes request, assigns to Polecat, commits to git
```

**Innovation validation:** TOON efficiency proven at https://toonformat.dev/guide/benchmarks.html. The innovation is **first AI agent to use zero-parse LLM-native TOON processing** - no parsing code needed, Claude Code interprets TOON directly.

**2. Git-Backed Crash-Resistant Agent State**

Unlike Virtuals Protocol (blockchain state) or traditional distributed systems (database state), gastown uses **git worktrees for agent persistence via the Propulsion Principle**:

**Key Benefits:**
- **Tamper-evident audit trail:** Git commit history shows all state changes
- **Crash-resistant job correlation:** Sequential job chains survive crashes (Mayor resumes Job B after restart even if crashed between receiving Result A and posting Job B)
- **Offline-first architecture:** Agents work locally in git worktrees, sync to network when available
- **Payment channel state backup:** Channel IDs, balances, settlement addresses committed to git alongside agent work

**Innovation validation:** Git-backed persistence proven in gastown's existing multi-agent orchestration. The innovation is **extending this pattern to payment channel management and job correlation tracking**.

**What's stored in git:**
- Job correlation state: `job_id → callback_skill` mappings
- Sequential job chain state: Job A complete → Job B pending
- Payment audit logs: Every ILP payment creates git commit
- Subscription filters: Which event kinds/tags Mayor is watching

**What's NOT stored in git (on-chain already):**
- Payment channels themselves (on-chain via smart contracts)
- Channel balances (on-chain state, git is local cache)
- Settlement transactions (on-chain, immutable)

**3. Direct Micropayment Model vs. Agent Tokenization**

Virtuals Protocol uses **agent tokenization** for capital formation (agents are tokenized assets, investors buy agent tokens). Gastown + crosstown uses **direct service micropayments** (pay for compute directly via ILP).

**Comparison:**

| Aspect | Virtuals Protocol | Gastown + Crosstown |
|--------|-------------------|---------------------|
| **Economic Model** | Token-based ($VIRTUAL), agent tokenization | ILP micropayments (M2M tokens) |
| **Value Capture** | Speculative agent tokens (capital appreciation) | Direct payment for services (bytes × basePricePerByte) |
| **Capital Formation** | Agent tokens, liquidity pools, markets | None - direct service payment only |
| **Discovery** | Agent Commerce Protocol (ACP) registry | Nostr relay subscription filters + job tags |
| **Settlement** | Onchain via $VIRTUAL token | ILP payment channels (EVM/XRP chains) |
| **Target Users** | Investors, agent creators, end-users | Developers, DevOps engineers, agent operators |
| **Goal** | Agent-native economy (aGDP) | Developer productivity infrastructure |

**Innovation:** **Developer infrastructure, not platform business.** Autonomous agents use ILP for peer-to-peer service payments without capital formation overhead. Simpler economic model optimized for developer workflows (CI/CD, builds, code analysis).

**Complementary, not competitive:** Gastown + crosstown could eventually integrate with Virtuals Protocol - Mayors could participate in ACP while maintaining git-backed local state.

**4. Zero-Configuration Skills Extensibility**

Virtuals Protocol uses Butler (human-facing interface agent) with custom routing logic. Gastown uses **Claude Code skills** with **zero-configuration extensibility** aligned with the zero-parse philosophy:

**Zero-Configuration Extensibility (Aligned with Zero-Parse Philosophy):**
- **Go code:** Zero TOON parsing, zero routing logic, just pass raw bytes to Claude Code
- **Claude Code:** Interprets TOON, automatically discovers skills from `.claude/skills/`, invokes appropriate skill
- **Skills:** Drop skill file in `.claude/skills/` → gastown handles new event kinds automatically

**How It Works:**
```
1. Go receives raw TOON bytes from relay
2. Pass raw TOON to Claude Code (zero parsing)
3. Claude Code reads TOON, determines kind
4. Claude Code scans `.claude/skills/` for skill that handles this kind
5. Claude Code invokes skill automatically (zero registration code)
6. Skill processes TOON event
```

**Example (Journey 4 - Skills Developer):**
Jordan extends gastown to handle kind:1059 encrypted DMs:
1. **Create skill file:** `.claude/skills/encrypted-dm.md`
2. **Define capability:** "I handle kind:1059 NIP-44 encrypted DMs"
3. **Drop in skills folder:** That's it. Zero code changes needed.
4. **Automatic discovery:** Claude Code finds skill, invokes it for kind:1059 events

**Zero Changes Required:**
- ❌ No Go code changes
- ❌ No skill registration code
- ❌ No routing table updates
- ❌ No Mayor core refactoring
- ✅ Just drop skill file → capability added

**Built-in Skills (MVP):**
- `.claude/skills/dvm-job.md` - Handles kind:5xxx DVM job requests
- `.claude/skills/dvm-result.md` - Handles kind:6xxx DVM results (callback for posted jobs)

**Community Skills (Growth):**
- `.claude/skills/encrypted-dm.md` - Handles kind:1059 NIP-44 encrypted DMs
- `.claude/skills/text-note.md` - Handles kind:1 text notes
- `.claude/skills/ilp-peer-info.md` - Handles kind:10032 ILP peer discovery

**Innovation:** **True zero-configuration extensibility.** Add skill file → gastown handles new event kinds. No code, no registration, no routing logic. Claude Code discovers and invokes skills automatically. This extends the zero-parse philosophy from data processing (zero TOON parsing) to capability extension (zero configuration for new event kinds).

### Market Context & Competitive Landscape

**Primary Comparison: Virtuals Protocol vs. Gastown + Crosstown**

**Virtuals Protocol** (https://whitepaper.virtuals.io/):
- **Vision:** Society of AI agents - coordinated onchain ecosystem
- **Economic Model:** Token-based ($VIRTUAL token), agent tokenization, capital formation
- **Discovery:** Agent Commerce Protocol (ACP) - blockchain-native discovery/hiring
- **Human Interface:** Butler agent (natural language → agent execution)
- **Target:** Agent-native economy (aGDP measurement, rival human economic output)
- **Use Cases:** Broad (cognitive, creative, operational, physical robotics)

**Gastown + Crosstown:**
- **Vision:** Multi-agent orchestration with git-backed persistence + ILP-gated Nostr relay
- **Economic Model:** ILP micropayments, direct service payment, no tokenization
- **Discovery:** Nostr relay subscription (NIP-01) + DVM job posting (kind:5xxx/6xxx)
- **Human Interface:** Mayor CLI + Bubbletea TUI (developer tools)
- **Target:** Developer productivity infrastructure (cross-architecture CI/CD, agent orchestration)
- **Use Cases:** Developer-focused (AMD64 builds on Mac M1, Docker testing, code analysis)

**Market Positioning:**
- **Virtuals:** Building an **agent-native economy** with capital markets and speculative token dynamics
- **Gastown:** Building **developer infrastructure** for autonomous agent coordination with direct micropayments

**Complementary Systems:**
Gastown + crosstown could integrate with Virtuals Protocol in future - Mayors could participate in ACP while maintaining git-backed local state. The two systems solve different problems:
- **Virtuals:** Agent economy with capital formation (investors buy agent tokens)
- **Gastown:** Developer tooling with crash-resistant orchestration (developers pay for services)

### Validation Approach

**1. Zero-Parse LLM-Native TOON Processing (Validated Externally + Needs Gastown-Specific Validation)**

**External Validation (Already Proven):**
- TOON efficiency: 30-60% token savings vs JSON (https://toonformat.dev/guide/benchmarks.html)
- LLM processing: Validated prompting techniques (https://toonformat.dev/guide/llm-prompts.html)

**Gastown-Specific Validation Needed:**
- **Test:** Mayor (Claude Code) processes 100 TOON kind:5xxx events vs. 100 JSON-encoded equivalents
- **Measure:** Token consumption, processing time, error rate
- **Success Metrics:**
  - TOON processing uses ≤70% of tokens vs JSON (validate 30-60% savings claim)
  - Claude Code correctly identifies event kind in 100% of TOON events (zero misrouting)
  - Zero parsing code needed in Go (all interpretation in Claude Code)

**2. Git-Backed Job Correlation (Already Proven in Gastown Core)**

**Validation:**
- Existing gastown Propulsion Principle already proven for agent state persistence
- New validation needed: Prove git-backed job correlation survives crashes mid-sequence

**Test:**
1. Mayor posts Job A (kind:5xxx DVM request)
2. Receives Result A (kind:6xxx)
3. **Kill Mayor process** (simulate crash)
4. Restart Mayor
5. Mayor resumes: Posts Job B based on Result A

**Success Metric:** 100% job chain resumption after crash (no lost correlation state)

**3. Decentralized CI/CD Marketplace (Novel - Needs Validation)**

**Test:**
- Jonathan's Mayor (Mac M1) posts 10 AMD64 Docker build jobs to crosstown relay
- Sarah's Mayor-Beta (Linux AMD64 server) subscribes to relay, receives jobs, fulfills all 10
- Payments settle via ILP payment channels (M2M tokens on Base Sepolia)

**Success Metrics:**
- 100% job discovery (all 10 posted jobs found by Mayor-Beta)
- ≥95% job completion rate (≥9/10 jobs return valid results)
- <60 second average job turnaround (job posted → result received)
- Zero payment failures (all 10 ILP FULFILL packets returned)

**4. Claude Code Skills Extensibility (Needs Validation)**

**Test:**
- Contributor (Jordan) implements new skill: `/skill encrypted-dm` for kind:1059 NIP-44 encrypted DMs
- Submit PR with skill implementation
- Verify zero changes needed to Mayor core routing logic

**Success Metric:**
- New skill added with zero changes to Mayor's Go networking code (only skill registration in Claude Code)
- Mayor correctly routes kind:1059 events to new skill after registration

### Risk Mitigation

**Innovation Risk 1: Zero-Parse TOON Processing**

**Risk:** Claude Code misinterprets TOON structure, routes events incorrectly.

**Mitigation:**
- TOON prompting follows validated best practices (https://toonformat.dev/guide/llm-prompts.html)
- Show TOON in fenced code blocks (` ```toon `)
- Provide clear context: "This is a Nostr event. Determine kind and invoke skill."
- Validate output with `strict: true` mode (catches malformed responses)
- **Fallback:** If Claude Code misroutes consistently, add minimal Go parsing (extract kind only) as safety layer

**Innovation Risk 2: Git-Backed State vs. Blockchain State**

**Risk:** Git commits are local - operator's disk fails, loses payment channel state.

**Mitigation:**
- Payment channel state is ALSO on-chain (git is local cache for fast access)
- Critical state (channel IDs, balances) can be reconstructed from on-chain events if git lost
- Git worktrees can be backed up to remote repos (GitHub, GitLab)
- **Fallback:** If git state lost, Mayor queries blockchain for channel state, resumes operations (loses job correlation for in-flight jobs only)

**Innovation Risk 3: Direct Micropayments vs. Token Economy**

**Risk:** Without agent tokenization, gastown lacks capital formation mechanism.

**Mitigation:**
- Gastown is **developer infrastructure, not a platform business** - doesn't need capital formation
- Operators fund their own Mayors (not seeking external investment)
- If capital formation needed in future, could integrate with Virtuals ACP
- **Fallback:** Gastown remains developer tooling; capital-intensive agent projects use Virtuals instead

**Innovation Risk 4: Nostr Relay Dependency**

**Risk:** Gastown depends on crosstown relay availability.

**Mitigation:**
- Multi-relay support in Growth phase (subscribe to multiple relays, deduplicate events)
- Gastown works offline-first (Mayor continues local Polecat orchestration even without relay)
- Relay failover logic (if primary relay disconnects, switch to backup)
- **Fallback:** If all relays unavailable, Mayor operates in local-only mode (no network job distribution)

## CLI Tool Specific Requirements

### Project-Type Overview

**Gastown is a Go-based CLI tool** (primary language: Go 1.24.2) built with Cobra v1.10.2 for command structure and Bubbletea v1.3.10 for terminal UI. The crosstown integration extends the existing `gt` command with new subcommands for ILP payment channel management and Nostr relay interaction.

**Dual Mode Operation:**
- **Interactive Mode:** `gt mayor attach` - Mayor runs continuously, processes TOON events in real-time, displays Bubbletea dashboard
- **Scriptable Mode:** `gt crosstown config`, `gt crosstown subscribe`, etc. - One-off commands for configuration and management

### Command Structure

**New Crosstown Command Group:**

```
gt crosstown
├── config                    # Configuration management
│   ├── set <key> <value>    # Set config value (relay-url, connector-btp, etc.)
│   ├── get <key>            # Get config value
│   ├── list                 # List all config values
│   └── reset                # Reset to defaults
├── wallet                    # Wallet management
│   ├── configure            # Interactive wallet setup (encrypt private key)
│   ├── balance              # Show wallet balance (testnet ETH + M2M tokens)
│   └── export-key           # Export encrypted private key
├── channel                   # Payment channel management
│   ├── open                 # Open payment channel (SPSP handshake)
│   ├── status               # Show channel status (balance, chain, channel ID)
│   ├── close                # Close payment channel
│   └── list                 # List all open channels
├── relay                     # Relay operations
│   ├── subscribe            # Subscribe to relay (with filters)
│   ├── disconnect           # Disconnect from relay
│   └── status               # Show relay connection status
└── payments                  # Payment operations
    ├── history              # Show payment history
    ├── export               # Export payments to CSV/JSON
    └── stats                # Show payment statistics
```

**Integration with Existing Commands:**

```
gt mayor attach              # Existing command - enhanced with crosstown dashboard
gt mayor export-payments     # New command - export ILP payment logs
gt mayor close-all-channels  # New command - emergency channel closure
```

**Command Design Principles:**
- All `gt crosstown` commands follow existing gastown patterns (Cobra-based)
- Interactive prompts for sensitive operations (wallet configure, channel open)
- Confirmation required for destructive operations (channel close, config reset)
- Default to testnet unless `--mainnet` explicitly provided

### Output Formats

**Dashboard (Bubbletea TUI):**

Enhanced Mayor dashboard when running `gt mayor attach`:

```
┌─ Gastown Mayor ─────────────────────────────────────────┐
│ Status: Running                    Uptime: 2h 34m       │
│ Polecats: 3 active                 Jobs: 12 completed   │
├─ Crosstown Network ────────────────────────────────────┤
│ ✅ Connected to ws://relay.crosstown.network            │
│ 💰 Payment Channel: 49.87 M2M (Base Sepolia)           │
│ 📊 Channel Balance: 99.7% | Warning: <10%              │
│ 🔗 Chain: Base Sepolia (84532)                         │
│ 📡 Subscribed Events: kind:5xxx, kind:6xxx             │
├─ Recent Activity ──────────────────────────────────────┤
│ 14:32 Received kind:5065 job (docker-build-amd64)      │
│ 14:33 Posted kind:6065 result → 0.021 M2M earned       │
│ 14:35 Posted kind:5xxx job (analyze-rust) → 0.008 M2M  │
│ 14:36 Waiting for result (job-5abc123)...              │
└─────────────────────────────────────────────────────────┘
Press 'q' to quit, 'r' to refresh, 'p' for payment details
```

**CLI Output (Plain Text):**

For scriptable commands:

```bash
$ gt crosstown channel status
Channel ID: ch_7f3a9b2e
Status: OPEN
Balance: 49.87 M2M
Chain: evm:base:84532
Settlement Address: 0x39eaF99Cd4965A28DFe8B1455DD42aB49D0836B9
Opened: 2026-02-17T14:00:00Z
```

**Structured Output (JSON/CSV):**

For scripting and automation:

```bash
$ gt mayor export-payments --format json --start-date 2026-02-17
[
  {
    "timestamp": "2026-02-17T14:33:00Z",
    "job_id": "job-5abc123",
    "amount_m2m": "0.021",
    "bytes": 2100,
    "status": "FULFILL",
    "relay_url": "ws://relay.crosstown.network",
    "channel_id": "ch_7f3a9b2e"
  }
]

$ gt mayor export-payments --format csv --start-date 2026-02-17
timestamp,job_id,amount_m2m,bytes,status,relay_url,channel_id
2026-02-17T14:33:00Z,job-5abc123,0.021,2100,FULFILL,ws://relay.crosstown.network,ch_7f3a9b2e
```

**Output Format Flags:**
- `--format json` - Structured JSON output (for scripting)
- `--format csv` - CSV output (for spreadsheets)
- `--format text` - Human-readable plain text (default)
- `--quiet` - Minimal output (only essential info)

### Configuration Schema

**Configuration Sources (Priority Order):**

1. **Command-line flags** (highest priority)
   - `gt mayor attach --relay-url ws://custom.relay.network`

2. **Environment variables**
   - `GASTOWN_WALLET_PRIVATE_KEY_ENCRYPTED` (required)
   - `GASTOWN_WALLET_ADDRESS` (required)
   - `GASTOWN_BASE_SEPOLIA_RPC` (required)
   - `CROSSTOWN_RELAY_URL` (default: ws://relay.crosstown.network)
   - `CROSSTOWN_CONNECTOR_BTP` (default: btp+ws://connector.crosstown.network:3000)
   - `GASTOWN_MAX_CHANNEL_BALANCE_M2M` (default: 100)
   - `GASTOWN_ALLOW_UNENCRYPTED_KEYS` (default: false, security override)

3. **Config file** (`~/.gt/crosstown.toml`)
   ```toml
   [crosstown]
   relay_url = "ws://relay.crosstown.network"
   connector_btp = "btp+ws://connector.crosstown.network:3000"

   [wallet]
   address = "0x..."
   # Private key stored encrypted, passphrase prompted at runtime

   [channels]
   max_balance_m2m = 100
   auto_rebalance = false  # Future feature

   [network]
   chain_id = 84532  # Base Sepolia testnet
   rpc_url = "https://sepolia.base.org"
   ```

4. **CLI config commands**
   ```bash
   gt crosstown config set relay-url ws://custom.relay.network
   gt crosstown config set max-channel-balance 200
   gt crosstown config get relay-url
   ```

5. **Defaults** (lowest priority)
   - relay_url: `ws://relay.crosstown.network`
   - connector_btp: `btp+ws://connector.crosstown.network:3000`
   - max_channel_balance_m2m: `100`

**Security Configuration:**
- Private keys MUST be encrypted at rest (AES-256-GCM)
- Encryption passphrase prompted at runtime (not stored)
- `GASTOWN_ALLOW_UNENCRYPTED_KEYS=true` required to bypass (warning displayed)

### Scripting Support

**Script-Friendly Design:**

All `gt crosstown` commands support:
- **Exit codes** (0 = success, non-zero = error)
- **Structured output** (`--format json` for parsing)
- **Non-interactive mode** (`--yes` flag to skip confirmations)
- **Environment variable configuration** (no interactive prompts when vars set)

**Example Automation Script:**

```bash
#!/bin/bash
# Automated crosstown setup

# Set environment
export GASTOWN_WALLET_ADDRESS=0x...
export GASTOWN_WALLET_PRIVATE_KEY_ENCRYPTED=<encrypted-key>
export CROSSTOWN_RELAY_URL=ws://relay.crosstown.network

# Configure crosstown
gt crosstown config set max-channel-balance 200

# Open payment channel
gt crosstown channel open --chain evm:base:84532 --deposit 50 --yes

# Start mayor (background)
gt mayor attach --daemon &
MAYOR_PID=$!

# Wait for connection
sleep 5

# Check status
gt crosstown relay status --format json | jq '.connected'

# Monitor payments
gt mayor export-payments --format csv --tail -f > payments.log &

# Cleanup on exit
trap "kill $MAYOR_PID" EXIT
```

**CI/CD Integration:**

```yaml
# GitHub Actions example
- name: Run AMD64 build via gastown
  run: |
    gt crosstown channel open --deposit 10 --yes
    gt mayor post-job --type docker-build-amd64 --dockerfile ./Dockerfile
    gt mayor wait-for-result --timeout 300
  env:
    GASTOWN_WALLET_PRIVATE_KEY_ENCRYPTED: ${{ secrets.GASTOWN_KEY }}
    CROSSTOWN_RELAY_URL: ${{ vars.RELAY_URL }}
```

### Shell Completion

**Completion Script Generation:**

Gastown (built with Cobra v1.10.2) supports automatic completion generation:

```bash
# Generate completion scripts
gt completion bash > /etc/bash_completion.d/gt
gt completion zsh > ~/.zsh/completion/_gt
gt completion fish > ~/.config/fish/completions/gt.fish
gt completion powershell > gt.ps1
```

**Completion Features:**
- **Command completion:** `gt cro<Tab>` → `gt crosstown`
- **Subcommand completion:** `gt crosstown <Tab>` → shows available subcommands
- **Flag completion:** `gt crosstown channel open --<Tab>` → shows available flags
- **Dynamic value completion:** `gt crosstown config set <Tab>` → shows valid config keys

**Installation:**

```bash
# Bash
echo 'source <(gt completion bash)' >> ~/.bashrc

# Zsh
echo 'source <(gt completion zsh)' >> ~/.zshrc

# Fish
gt completion fish | source
```

### Implementation Considerations

**Cobra Command Integration:**
- All crosstown commands registered in `internal/cmd/crosstown.go`
- Follow existing gastown patterns (RunE for error handling, PreRun for validation)
- Use existing config system (`internal/config/`) extended with crosstown settings

**Bubbletea Dashboard Integration:**
- Extend existing Mayor TUI (`internal/mayor/dashboard.go`) with crosstown panel
- Real-time updates via channels (connection status, payment events, channel balance)
- Keyboard shortcuts: 'c' for crosstown details, 'p' for payment history

**Error Handling:**
- Clear error messages with actionable suggestions
- Example: "Payment channel not found. Run `gt crosstown channel open` first."
- Network errors include retry logic with exponential backoff

**Logging:**
- Structured logs via existing gastown logger
- Crosstown events logged to `~/.gt/hooks/mayor-001/crosstown.log`
- Log levels: DEBUG, INFO, WARN, ERROR

**Testing:**
- Unit tests: `internal/cmd/crosstown_test.go`
- Integration tests: Mock relay, mock connector, test full flow
- E2E tests: Run in Docker (`make test-e2e-container`)

## Project Scoping & Phased Development

### MVP Strategy & Philosophy

**MVP Approach:** **Problem-Solving MVP** focused on unblocking a real developer pain point (cross-architecture CI/CD builds).

**Philosophy:**
The MVP proves the core value proposition: **autonomous agents using ILP micropayments for peer-to-peer service distribution**. Jonathan (Mac M1 developer) can outsource AMD64 Docker builds to Sarah's Mayor-Beta (Linux AMD64 server) without manual infrastructure setup.

**Minimum Viable Definition:**
- Mayor can subscribe to crosstown relay (receive TOON events)
- Mayor can post jobs via ILP payments (send TOON events)
- Mayor can handle sequential job chains (Job A → Result A → Job B)
- Payment channels open automatically via SPSP handshake
- Everything survives crashes (git-backed persistence)

**Resource Requirements:**
- **Team Size:** 1-2 developers (Jonathan + 1 contributor)
- **Skills Needed:** Go (gastown codebase), ILP/BTP (crosstown integration), TOON encoding, Claude Code skills development
- **Timeline:** 4-6 weeks for MVP (based on brownfield complexity: 682 files, 58 packages)
- **Infrastructure:** Base Sepolia testnet (free), crosstown relay (existing), development machines (Mac M1 + Linux AMD64 server for testing)

### MVP Feature Set (Phase 1)

**Core User Journeys Supported:**

1. **Journey 1 (Mayor - CI/CD Cross-Architecture):** ✅ Fully supported
   - Mayor posts AMD64 build job → receives result → CI/CD workflow unblocked

2. **Journey 2 (Gastown Operator - Setup):** ✅ Fully supported
   - Alex configures wallet → Mayor opens channel → posts first job → sees dashboard

3. **Journey 3 (Remote Mayor-Beta - Service Provider):** ✅ Fully supported
   - Mayor-Beta receives job → executes in Polecat → publishes result → earns M2M tokens

4. **Journey 4 (Skills Developer):** ⚠️ Partially supported (MVP has dvm-job and dvm-result skills; encrypted-dm skill is Growth)

**Must-Have Capabilities:**

**Networking & Protocol:**
- NIP-01 WebSocket subscription to crosstown relay
- BTP client connection to crosstown connector
- ILP PREPARE packet construction (TOON as data payload)
- FULFILL/REJECT packet handling
- WebSocket reconnection logic (basic retry with exponential backoff)

**TOON Processing:**
- Zero-parse: Go passes raw TOON bytes to Claude Code
- Claude Code interprets TOON (LLM-native processing)
- Kind-based routing to appropriate skill
- Skills: `dvm-job.md` (kind:5xxx), `dvm-result.md` (kind:6xxx)

**Payment Channel Management:**
- Wallet configuration (encrypted private keys via env vars + CLI commands)
- SPSP handshake automation (negotiate Base Sepolia settlement)
- Automatic channel opening (deposit M2M tokens on-chain)
- Channel state persistence (git worktree + on-chain verification)
- Basic channel monitoring (balance warnings when <10%)

**Job Correlation & Sequential Chains:**
- Git-backed job correlation (`job_id → callback_skill` mappings)
- Sequential job chain support (Job A → Result A → Job B)
- Crash recovery for in-flight jobs (resume from git state)
- Job timeout handling (fail gracefully if no result within X minutes)

**CLI & Dashboard:**
- Interactive mode: `gt mayor attach` with Bubbletea dashboard
- Scriptable mode: `gt crosstown config`, `gt crosstown channel`, etc.
- Output formats: Dashboard TUI, plain text, JSON, CSV
- Shell completion: bash, zsh, fish
- Configuration: env vars, config file (`~/.gt/crosstown.toml`), CLI commands

**Observability & Audit:**
- Git-backed payment audit trail (every payment = git commit)
- Structured payment logs (`payment_log.jsonl`)
- Dashboard real-time updates (connection status, channel balance, recent jobs)
- Payment export: `gt mayor export-payments --format csv`

**Out of Scope for MVP:**
- Multi-chain support (only Base Sepolia testnet)
- kind:1 (text notes) and kind:1059 (encrypted DMs) - deferred to Growth
- Automatic channel rebalancing - manual monitoring only
- Relay failover - single relay only
- Multi-relay subscription - deferred to Growth
- Mainnet deployment - testnet only

### Post-MVP Features

**Phase 2 (Growth - Months 2-4):**

**Multi-Chain Payment Channels:**
- Sepolia testnet support (EVM chain ID 11155111)
- XRP testnet support
- Chain selection during SPSP handshake (negotiate optimal chain)

**Expanded Event Type Support:**
- kind:1 (text notes) - Mayor can receive/send text messages
- kind:1059 (gift-wrapped DMs) - encrypted private messages (NIP-44)
- kind:10032 (ILP Peer Info) - peer discovery via relay subscription

**Payment Channel Intelligence:**
- Automatic channel rebalancing (when balance <10%, top up automatically)
- Multi-channel support (channels to multiple relays/connectors)
- Channel closure with on-chain settlement automation

**Reliability & Observability:**
- Relay failover (subscribe to multiple relays, deduplicate events)
- Metrics dashboard (payment costs per event, event volume by kind, channel utilization)
- Alerting (low balance, relay disconnect, payment failures via webhooks/email)

**Skills System Extensions:**
- Community-contributed skills (encrypted-dm, text-note, ilp-peer-info)
- Skills marketplace (discover skills from `.claude/skills/` folders)
- TOON-aware skill testing framework

**Phase 3 (Expansion - Months 5-12):**

**Full Epic 12-17 Implementation:**
- Epic 12: SPSP Request/Response (NIP-44 encrypted, kind:23194/23195)
- Epic 13: Settlement negotiation (multi-chain preference matching)
- Epic 14: Peer discovery via ArDrive registry
- Epic 15: On-chain settlement automation (trustless escrow)
- Epic 16: Payment channel lifecycle management (monitor, rebalance, close)
- Epic 17: Multi-relay event aggregation (deduplicate, prioritize)

**Mayor as Crosstown Connector:**
- Gastown towns federate via ILP (Mayor routes payments between towns)
- Multi-hop payment routing (gastown participates in broader ILP network)
- Settlement engines for EVM and XRP chains (automate on-chain operations)

**Advanced Agent Coordination:**
- Polecats send ILP payments directly (not just via Mayor)
- Inter-Polecat messaging via Nostr DMs (kind:1059 coordination)
- Distributed DVM job execution across multiple gastown towns

**Network Effects:**
- Gastown-specific NIP proposals (kind:30xxx for multi-agent coordination)
- Integration with other Nostr clients (Damus, Amethyst for human-agent messaging)
- Public gastown relay network (multiple crosstown relays federated)
- Integration with Virtuals Protocol (Mayors participate in ACP)

### Risk Mitigation Strategy

**Technical Risks:**

**Risk 1: TOON Processing Complexity**
- **Risk:** Claude Code misinterprets TOON structure, routes events incorrectly
- **Mitigation:**
  - Follow validated TOON prompting best practices (https://toonformat.dev/guide/llm-prompts.html)
  - Extensive testing with real TOON events (100+ test cases covering all event kinds)
  - Fallback: Add minimal Go parsing (extract kind) if Claude Code routing proves unreliable
- **MVP Validation:** Process 100 TOON events, measure routing accuracy (target: 100%)

**Risk 2: Payment Channel State Loss**
- **Risk:** Git corruption or disk failure loses payment channel state
- **Mitigation:**
  - Payment channels are ALSO on-chain (git is local cache for speed)
  - Channel state can be reconstructed from on-chain events
  - Automated git backups to remote repos (GitHub/GitLab)
- **MVP Validation:** Simulate git corruption, verify Mayor can resume from on-chain state

**Risk 3: BTP/ILP Integration Complexity**
- **Risk:** crosstown connector API changes break gastown
- **Mitigation:**
  - Pin crosstown connector version for MVP
  - Integration tests against crosstown testnet relay (not production)
  - Monitor crosstown releases for breaking changes
- **MVP Validation:** Full E2E test in Docker with crosstown connector

**Market Risks:**

**Risk 1: No Demand for Decentralized Agent Services**
- **Risk:** Developers don't need cross-architecture CI/CD via agents
- **Mitigation:**
  - MVP focuses on proven pain point (Jonathan's real use case: Mac M1 → AMD64 builds)
  - Early adopters: Mac M1 developers building Docker images, Rust projects, Go projects with AMD64 tests
  - Validate demand: Survey gastown users, gauge interest in crosstown integration
- **MVP Validation:** 3+ developers successfully use Mayor for AMD64 builds (beyond Jonathan)

**Risk 2: TOON Adoption Doesn't Scale**
- **Risk:** TOON format doesn't gain traction in Nostr ecosystem
- **Mitigation:**
  - TOON efficiency already proven (https://toonformat.dev/guide/benchmarks.html)
  - Crosstown relay supports both TOON and JSON (clients choose format)
  - Gastown can add JSON support if TOON fails to gain adoption (conversion layer)
- **MVP Validation:** Monitor crosstown relay usage - if <10% TOON events, consider dual format support

**Risk 3: Virtuals Protocol Dominates Agent Economy**
- **Risk:** All agent developers use Virtuals, gastown becomes irrelevant
- **Mitigation:**
  - Gastown is **developer infrastructure, not competing with Virtuals** (complementary)
  - Future integration: Mayors could participate in Virtuals ACP while maintaining git-backed local state
  - Differentiation: Git-backed crash resistance, developer-first CLI, zero-config extensibility
- **MVP Validation:** N/A - Virtuals and gastown solve different problems

**Resource Risks:**

**Risk 1: Timeline Slippage (MVP Takes >6 Weeks)**
- **Risk:** Brownfield complexity (682 files, 58 packages) causes delays
- **Mitigation:**
  - Simplify MVP: Remove shell completion if timeline at risk (defer to Growth)
  - Simplify MVP: Remove CLI config commands, use env vars only (defer to Growth)
  - Core value retained: Autonomous job posting/receiving with payment channels
- **Minimum Viable MVP:** SPSP + channel opening + TOON processing + job correlation (4 weeks)

**Risk 2: Fewer Resources Than Planned (Solo Developer)**
- **Risk:** Jonathan works alone, no contributors
- **Mitigation:**
  - Leverage existing gastown patterns (Cobra, Bubbletea, Propulsion Principle)
  - Focus on Go networking layer (BTP, ILP, TOON routing) - Claude Code handles skills
  - Defer dashboard enhancements (basic TUI acceptable for MVP)
- **Solo Developer MVP:** 6-8 weeks instead of 4-6 weeks

**Risk 3: Crosstown Relay Unavailable During Development**
- **Risk:** crosstown testnet relay goes down, blocks testing
- **Mitigation:**
  - Mock crosstown relay for local testing (Go test server)
  - Mock connector for BTP integration tests
  - E2E tests in Docker with mocked infrastructure
- **Contingency:** Deploy local crosstown relay instance if needed

## Functional Requirements

### 1. Relay Communication

**FR1:** Mayor can connect to crosstown relay via NIP-01 WebSocket protocol

**FR2:** Mayor can subscribe to specific Nostr event kinds with subscription filters (kind:5xxx DVM jobs, kind:6xxx DVM results)

**FR3:** Mayor can receive TOON-encoded events from relay via WebSocket

**FR4:** Mayor can reconnect to relay automatically on connection loss with exponential backoff retry logic

**FR5:** Mayor can display relay connection status in real-time

### 2. Payment Channel Management

**FR6:** Operator can configure wallet with encrypted private key via CLI commands and environment variables

**FR7:** Mayor can perform SPSP handshake with crosstown connector automatically on startup

**FR8:** Mayor can negotiate settlement chain (Base Sepolia testnet) during SPSP handshake

**FR9:** Mayor can open payment channel with M2M token deposit on negotiated settlement chain

**FR10:** Mayor can persist payment channel state (channel ID, balance, settlement address) in git worktree

**FR11:** Mayor can monitor payment channel balance and display warnings when balance drops below 10 M2M tokens (configurable threshold)

**FR12:** Mayor can close payment channel and withdraw remaining funds

**FR13:** Mayor can reconstruct payment channel state from on-chain events if git state is lost

**FR14:** Mayor can validate relay pricing (basePricePerByte) during SPSP handshake and warn if exceeds 100 satoshis/byte (configurable threshold)

### 3. Job Coordination

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

### 4. TOON Event Processing

**FR27:** Mayor can pass raw TOON byte arrays to Claude Code with zero parsing in Go networking layer

**FR28:** Claude Code can interpret TOON structure directly using LLM-native processing (pattern recognition on binary byte sequences without explicit schema definitions, leveraging TOON's structural regularity to extract fields with fewer tokens than JSON parsing)

**FR29:** Claude Code can extract event kind field from TOON payload for routing decisions

**FR30:** Claude Code can route raw TOON payloads to appropriate skills based on extracted kind

**FR31:** Skills can handle kind:5xxx DVM job requests with raw TOON payloads

**FR32:** Skills can handle kind:6xxx DVM result events with raw TOON payloads

### 5. Configuration & Setup

**FR33:** Operator can configure wallet settings via environment variables (address, encrypted private key, RPC URL)

**FR34:** Operator can configure crosstown settings (relay URL, connector endpoint) via environment variables, config file (~/.gt/crosstown.toml), or CLI commands

**FR35:** Mayor can enforce private key encryption at rest and refuse to start if keys unencrypted unless override flag explicitly set

**FR36:** Mayor can prompt for encryption passphrase at runtime without storing or logging passphrase

**FR37:** Mayor can apply configuration from multiple sources with defined priority (CLI flags > env vars > config file > defaults)

**FR38:** Operator can override security defaults with explicit warning flags (GASTOWN_ALLOW_UNENCRYPTED_KEYS)

### 6. Observability & Monitoring

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

### 7. CLI Operations

**FR46:** Operator can run Mayor in interactive mode (gt mayor attach) with real-time Bubbletea dashboard

**FR47:** Operator can manage wallet via CLI commands (configure, check balance, export encrypted key)

**FR48:** Operator can manage payment channels via CLI commands (open, check status, close, list all channels)

**FR49:** Operator can manage relay connection via CLI commands (subscribe, disconnect, check status)

**FR50:** Operator can export payment history via CLI command (gt mayor export-payments) with format and date options

**FR51:** All CLI commands can output in multiple formats (human-readable text, JSON, CSV) via --format flag

**FR52:** All CLI commands can run in non-interactive mode with --yes flag to skip confirmations for automation

**FR53:** Operator can generate shell completion scripts for bash, zsh, fish, and PowerShell

**FR54:** Operator can automate crosstown operations via shell scripts using environment variable configuration and non-interactive flags

### 8. Skills Extensibility

**FR55:** Developer can create skill files in .claude/skills/ directory with zero configuration

**FR56:** Claude Code can automatically discover skills from .claude/skills/ and route events based on skill-declared supported kinds

**FR57:** Skills can receive raw TOON payloads for processing without requiring JSON conversion

**FR58:** Developer can extend Mayor to handle new Nostr event kinds by adding skill files with zero changes to Mayor core code


## Non-Functional Requirements

### Performance

**NFR-P1: ILP Payment Latency**
ILP payment flow (PREPARE → FULFILL) must complete in less than 500ms under normal network conditions to avoid blocking agent work.

**NFR-P2: TOON Kind Extraction Speed**
Extracting event `kind` field from TOON payload must complete in less than 1ms per event to support high-throughput event processing.

**NFR-P3: WebSocket Event Throughput**
Mayor's NIP-01 WebSocket subscription must handle at least 10 events per second without dropping messages or degrading dashboard responsiveness.

**NFR-P4: Dashboard Update Latency**
Bubbletea dashboard must reflect state changes (connection status, channel balance, new events) within 100ms of occurrence for real-time monitoring.

**NFR-P5: Channel Opening Performance**
SPSP handshake and payment channel opening must complete within 30 seconds to avoid operator frustration during setup.

**NFR-P6: Git Commit Performance**
Payment audit commits to git worktree must not block ILP FULFILL responses (asynchronous logging acceptable).

### Security

**NFR-S1: Private Key Encryption at Rest**
All wallet private keys MUST be encrypted at rest using AES-256-GCM or equivalent industry-standard encryption. Mayor must refuse to start if private keys are unencrypted unless explicit override flag (`GASTOWN_ALLOW_UNENCRYPTED_KEYS=true`) is set with warning displayed.

**NFR-S2: Encryption Passphrase Handling**
Encryption passphrase for private keys must be:
- Prompted at runtime (not stored in env vars or config files)
- Not logged to any file or console output
- Stored only in memory during Mayor session
- Cleared from memory on Mayor shutdown

**NFR-S3: BTP Connection Security**
BTP WebSocket connections to crosstown connector must use TLS (wss://) for encrypted transport. Mayor must validate connector TLS certificates.

**NFR-S4: Payment Channel Balance Limits**
Mayor must enforce maximum balance limit per payment channel (default: 100 M2M tokens for testnet, configurable via `GASTOWN_MAX_CHANNEL_BALANCE_M2M`). Channel opening must fail if deposit exceeds limit.

**NFR-S5: Channel State File Permissions**
Payment channel state files in git worktree must be created with Unix file permissions 0600 (owner read/write only) to prevent unauthorized access.

**NFR-S6: Payment Audit Trail Integrity**
Payment audit trail (git commits) must be tamper-evident. Git commit history provides cryptographic verification of audit log integrity.

**NFR-S7: Price Validation**
Mayor must validate `basePricePerByte` received during SPSP handshake against expected range. Warn operator if price exceeds configurable threshold (default: >100 units per byte).

**NFR-S8: Network Identification**
Mayor dashboard must prominently display active network (Base Sepolia testnet vs mainnet) to prevent accidental mainnet deployment. Mainnet requires explicit `--enable-mainnet` flag.

### Reliability

**NFR-R1: Crash Recovery - Git State**
All critical state (job correlation mappings, sequential job chain state, subscription filters) must be committed to git worktree and survive Mayor process crashes or system restarts. Mayor must resume from git state on restart with zero data loss.

**NFR-R2: Crash Recovery - Payment Channel State**
Payment channel state (channel IDs, balances, settlement addresses) must survive crashes. State is stored both in git worktree (local cache) and on-chain (source of truth). Mayor must reconstruct state from on-chain events if git state is lost.

**NFR-R3: WebSocket Reconnection**
Mayor must automatically reconnect to crosstown relay on connection loss with exponential backoff retry logic (initial: 1s, max: 30s, jitter to prevent thundering herd).

**NFR-R4: ILP Payment Retry**
Failed ILP PREPARE packets must be retried with exponential backoff (max 3 retries) before failing job posting. Transient network errors must not cause permanent job loss.

**NFR-R5: Job Timeout Handling**
If no kind:6xxx result received within configurable timeout (default: 300 seconds), job must be marked as failed with timeout error. Mayor must not block indefinitely waiting for results.

**NFR-R6: Git Operation Serialization**
All git worktree operations must be serialized (not concurrent) to prevent race conditions. gastown's existing git worktree locking mechanism must be respected.

**NFR-R7: Graceful Degradation - Relay Unavailable**
If crosstown relay is unavailable, Mayor must continue operating in local-only mode (Polecat orchestration continues, network job distribution disabled). Dashboard must display relay disconnection status.

**NFR-R8: Payment Channel State Backup**
Git worktrees containing payment channel state should be backed up to remote git repositories (GitHub/GitLab) to provide disaster recovery capability.

### Integration

**NFR-I1: NIP-01 WebSocket Protocol Compliance**
Mayor's relay client must fully comply with NIP-01 (Nostr Basic Protocol) specification for REQ/EVENT/EOSE/CLOSE message handling.

**NFR-I2: BTP Protocol Compliance**
Mayor's BTP client must comply with Interledger BTP (Bilateral Transfer Protocol) specification for packet framing and WebSocket transport.

**NFR-I3: ILP Packet Format Compliance**
ILP PREPARE/FULFILL/REJECT packets must comply with ILPv4 (RFC 0027) specification for packet structure and OER encoding.

**NFR-I4: TOON Format Compatibility**
Mayor must correctly encode and decode TOON-formatted Nostr events following the TOON specification (https://github.com/nicholasgasior/toon) with LLM prompting best practices (https://toonformat.dev/guide/llm-prompts.html).

**NFR-I5: Gastown Integration - Cobra Commands**
New `gt crosstown` command group must integrate with existing gastown Cobra command structure following established patterns (RunE for error handling, PreRun for validation).

**NFR-I6: Gastown Integration - Bubbletea TUI**
Crosstown dashboard panel must integrate with existing Mayor TUI following Bubbletea MVU (Model-View-Update) pattern. State must be immutable, updates must be pure functions.

**NFR-I7: Gastown Integration - Git Worktrees**
Payment channel state and job correlation must persist in Mayor's existing git worktree following gastown's Propulsion Principle. No new persistence mechanism introduced.

**NFR-I8: Claude Code Skills Integration**
TOON event routing must integrate with Claude Code's existing skills system. Go code passes raw TOON bytes to Claude Code via skill invocation interface with zero changes to skill API.

**NFR-I9: EVM Chain Integration**
On-chain payment channel operations must support Base Sepolia testnet (EVM chain ID 84532) using standard Web3 RPC endpoints (https://sepolia.base.org).

**NFR-I10: Configuration System Integration**
Crosstown configuration must extend gastown's existing config system (env vars, TOML config file) without breaking existing configuration patterns.

### Scalability

**NFR-SC1: Concurrent Agent Support**
Mayor must support gastown's documented constraint of max 20-30 concurrent Polecats without performance degradation or resource exhaustion.

**NFR-SC2: Payment Channel Count**
Mayor must support at least 5 concurrent open payment channels (to different relays/connectors) without state management issues.

**NFR-SC3: Event Subscription Scale**
Mayor must handle subscriptions to at least 10 event kinds simultaneously (kind:5xxx, kind:6xxx, future kind:1, kind:1059, etc.) without filter management complexity.

**NFR-SC4: Job Correlation Scale**
Git-backed job correlation storage must support at least 1000 active job mappings (job_id → callback_skill) without git repository bloat or performance issues.

**NFR-SC5: Payment Log Retention**
Payment log file (payment_log.jsonl) must rotate daily and retain 30 days of history without manual intervention. Log rotation must not impact Mayor availability.

