# Implementation Patterns & Consistency Rules

## Pattern Categories Defined

**Critical Conflict Points Identified:** 7 areas where AI agents could make different implementation choices without explicit guidance.

**Patterns Established:**
1. Package & Type Naming
2. Constant & Error Naming
3. Git Commit Message Format (Audit Trail)
4. Bubbletea Message Naming (MVU Pattern)
5. Environment Variable Naming
6. CLI Error Message Format
7. Structured Logging Format

---

## Naming Patterns

### Package & Type Naming

**Package Names (established):**
```go
internal/ilp/          // ILP/BTP protocol implementation
internal/nostr/        // NIP-01 WebSocket + TOON encoding
internal/crosstown/    // Integration layer, channel manager, job correlation
```

**Struct Naming Convention: Brief (package provides context)**
```go
// ✅ CORRECT (package crosstown)
type ChannelManager struct {
    channels map[string]*ChannelState
}

type ChannelState struct {
    ID              string
    Status          Status
    BalanceM2M      string
    SettlementChain string
}

// ❌ INCORRECT (redundant with package name)
type CrosstownChannelManager struct {}
type PaymentChannelState struct {}
```

**Interface Naming Convention: Full descriptive**
```go
// ✅ CORRECT (clear across package boundaries)
package nostr
type NostrRelayClient interface {
    Subscribe(filters []Filter) error
    Publish(event []byte) error
}

package ilp
type ILPConnectorClient interface {
    SendPrepare(packet []byte) ([]byte, error)
    Handshake() error
}

// ❌ INCORRECT (ambiguous when imported)
type RelayClient interface {}
type Client interface {}
```

**Rationale:**
- Structs are brief because package name provides context (`crosstown.ChannelManager` is clear)
- Interfaces are fully descriptive because they cross package boundaries and are used for mocking
- Follows Go stdlib conventions (e.g., `http.Client` vs `http.Handler`)

---

### Constant & Error Naming

**Constant Naming: Full prefix (Go stdlib style)**
```go
// ✅ CORRECT
package crosstown

type Status string

const (
    ChannelStatusOpen    Status = "open"
    ChannelStatusClosing Status = "closing"
    ChannelStatusClosed  Status = "closed"
)

type EventKind int

const (
    EventKindDVMRequest EventKind = 5000
    EventKindDVMResult  EventKind = 6000
)

// ❌ INCORRECT (ambiguous)
const (
    Open    Status = "open"  // What's open? Channel? Connection?
    Closing Status = "closing"
)
```

**Error Variable Naming: Err prefix (Go convention)**
```go
// ✅ CORRECT
package crosstown

var (
    ErrChannelClosed     = errors.New("payment channel is closed")
    ErrInsufficientFunds = errors.New("insufficient channel balance")
    ErrInvalidTOON       = errors.New("invalid TOON encoding")
    ErrChannelNotFound   = errors.New("payment channel not found")
)

package nostr

var (
    ErrRelayDisconnected = errors.New("relay WebSocket disconnected")
    ErrInvalidEventKind  = errors.New("invalid Nostr event kind")
)

// ❌ INCORRECT (not idiomatic Go)
var (
    CrosstownErrChannelClosed = errors.New("...")
    ChannelClosedError        = errors.New("...")
)
```

**Rationale:**
- Full prefix for constants eliminates ambiguity (matches Go stdlib: `http.StatusOK`, `os.O_RDONLY`)
- `Err` prefix for error variables is idiomatic Go (matches stdlib: `io.EOF`, `sql.ErrNoRows`)

---

## Structure Patterns

### File Organization (follows existing gastown patterns)

**From project-context.md (already established):**
```
internal/crosstown/
  channel_manager.go           # Implementation
  channel_manager_test.go      # Unit tests (gomock)
  types.go                     # Types, constants, errors
  config.go                    # Configuration structs
  integration_test.go          # Integration tests (build tag: integration)
```

**New crosstown-specific files:**
```
internal/ilp/
  client.go                    # BTP client implementation
  client_test.go               # Unit tests
  rpc.go                       # Base Sepolia RPC client
  rpc_test.go                  # RPC tests
  spsp.go                      # SPSP handshake handler
  spsp_test.go                 # SPSP tests
  types.go                     # ILP types, constants, errors

internal/nostr/
  relay.go                     # NIP-01 WebSocket client
  relay_test.go                # Unit tests
  encoding.go                  # TOON encoding (using toon-go)
  encoding_test.go             # TOON encoding tests
  types.go                     # Nostr types, event kinds

internal/crosstown/
  channel_manager.go           # Payment channel manager
  channel_manager_test.go      # Unit tests
  job_tracker.go               # Job correlation tracker
  job_tracker_test.go          # Job correlation tests
  wallet.go                    # Wallet, key derivation (Argon2)
  wallet_test.go               # Wallet tests
  types.go                     # Crosstown types, constants
  e2e_test.go                  # E2E tests (full flow)
```

**Generated Mocks:**
```
internal/ilp/mocks/
  mock_connector_client.go     # Generated by gomock

internal/nostr/mocks/
  mock_relay_client.go         # Generated by gomock
```

---

## Format Patterns

### Git Commit Message Format (Audit Trail)

**Pattern: Audit-focused with structured data**

**Template:**
```
crosstown: <action> <resource> <identifier>

<Key>: <Value>
<Key>: <Value>
...

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Examples:**

**Channel opened:**
```
crosstown: payment channel ch_7f3a9b2e opened

Deposit: 50.00 M2M
Chain: evm:base:84532
Connector: btp+ws://connector.crosstown.network:3000
Settlement Address: 0x39eaF99Cd4965A28DFe8B1455DD42aB49D0836B9

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Balance updated:**
```
crosstown: payment channel ch_7f3a9b2e balance updated

Previous: 50.00 M2M
Current: 49.87 M2M
Job ID: job-5abc123
Payment: 0.13 M2M (1300 bytes × basePricePerByte)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Channel closed:**
```
crosstown: payment channel ch_7f3a9b2e closed

Final Balance: 49.87 M2M
Withdrawal Address: 0x...
On-chain TX: 0x...

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Rationale:**
- Structured format enables audit log parsing and analysis
- Co-Authored-By provides AI agent attribution (gastown pattern)
- Tamper-evident via git commit history (NFR-S6)

---

### Environment Variable Naming

**Pattern: GASTOWN_CROSSTOWN_* prefix (some GASTOWN_* for wallet)**

**All crosstown configuration:**
```bash