# Non-Functional Requirements

## Performance

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

## Security

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

## Reliability

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

## Integration

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

## Scalability

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

