# Core Architectural Decisions

## Decision Priority Analysis

**Critical Decisions (Block Implementation):**
All 5 decisions below must be implemented for MVP to function:

1. **WebSocket & Networking Libraries** - Required for relay communication (FR1-FR5)
2. **TOON Encoding Strategy** - Required for creating DVM events (FR15, FR25)
3. **Payment Channel State Storage** - Required for crash recovery (NFR-R1, NFR-R2)
4. **Encryption Key Management** - Required for security compliance (NFR-S1, NFR-S2)
5. **Testing & Mocking Strategy** - Required for ≥50% coverage and E2E validation

**Deferred Decisions (Post-MVP):**
- OS Keyring integration (defer to mainnet preparation)
- Multi-chain payment channel support (defer to Growth - Epic 13)
- Advanced monitoring/alerting infrastructure (defer to Growth)
- Performance optimization (tuning after MVP validation)

## Networking & Protocol Libraries

**Decision: gorilla/websocket + net/http (stdlib)**

**Rationale:**
- gorilla/websocket: Battle-tested WebSocket client with ~20K stars, production-proven for fintech applications
- net/http: Standard library HTTP client, zero dependencies, sufficient for Base Sepolia RPC calls
- Reliability over convenience: Matches NFR-R3 (exponential backoff) and NFR-R4 (ILP retry) requirements

**Version:**
- gorilla/websocket: Latest stable (verify at implementation)
- net/http: Go 1.24.2 stdlib

**Affects:**
- `internal/nostr/relay.go` - NIP-01 WebSocket client for relay subscription
- `internal/ilp/rpc.go` - HTTP client for Base Sepolia RPC calls (channel opening, state queries)

**Implementation Notes:**
- WebSocket: Implement reconnection with exponential backoff (1s→30s, jitter for thundering herd)
- HTTP: Implement retry logic with backoff for transient RPC failures
- Both: Proper context.Context handling for cancellation and timeouts

---

## TOON Encoding & Decoding

**Decision: github.com/ALLiDoizCode/toon-go (fork with upstream contribution)**

**Rationale:**
- Full TOON spec compliance from day one
- Opportunity to contribute improvements back to official library
- Supports crosstown ecosystem development
- Eliminates custom implementation maintenance burden

**Version:**
- Use fork: github.com/ALLiDoizCode/toon-go (latest)
- Contribute improvements upstream to official toon-go repository

**Affects:**
- `internal/nostr/encoding.go` - TOON encoding for kind:5xxx (DVM job requests) and kind:6xxx (DVM results)
- `internal/crosstown/events.go` - Event construction with TOON payloads

**Implementation Notes:**
- Zero-parse on receiving side: Go passes raw TOON bytes to Claude Code (no Go decoding needed)
- Encoding needed for creating events: Mayor posts jobs, publishes results
- Integration work may surface library improvements (upstream contribution opportunity)

---

## Payment Channel State Storage

**Decision: One TOML file per channel in git worktree**

**Structure:**
```
~/.gt/hooks/mayor-001/crosstown/channels/
  ch_7f3a9b2e.toml
  ch_abc456f.toml
```

**File Format (TOML):**
```toml
channel_id = "ch_7f3a9b2e"
status = "open"  # open, closing, closed
balance_m2m = "49.87"
settlement_chain = "evm:base:84532"
settlement_address = "0x39eaF99Cd4965A28DFe8B1455DD42aB49D0836B9"
opened_at = "2026-02-17T14:00:00Z"
connector_btp = "btp+ws://connector.crosstown.network:3000"
relay_url = "ws://relay.crosstown.network"
```

**Rationale:**
- Consistent with gastown's TOML configuration pattern
- Human-readable for debugging and manual inspection
- One git commit per channel state change = tamper-evident audit trail (NFR-S6)
- Simple file-per-channel cleanup when channels close
- Easy to implement on-chain recovery (read TOML, query blockchain)

**Git Commit Pattern:**
- Every balance update, status change, or channel operation = one commit
- Commit message includes: channel ID, operation type, amount (if applicable)
- Example: `"Payment channel ch_7f3a9b2e: balance update 50.00 → 49.87 M2M (job-5abc123)"`

**Affects:**
- `internal/crosstown/channel_manager.go` - Channel state persistence
- Git commits for audit trail (FR40, NFR-S6)

**File Permissions:**
- 0600 (owner read/write only) per NFR-S5

---

## Encryption & Key Management

**Decision: Argon2 key derivation + environment variable storage**

**Key Derivation:**
- Algorithm: Argon2id (golang.org/x/crypto/argon2)
- Parameters: Time=1, Memory=64MB, Threads=4, KeyLen=32 (AES-256)
- Salt: Generated per encrypted key, stored alongside ciphertext

**Encrypted Key Storage:**
```bash