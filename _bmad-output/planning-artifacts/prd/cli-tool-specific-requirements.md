# CLI Tool Specific Requirements

## Project-Type Overview

**Gastown is a Go-based CLI tool** (primary language: Go 1.24.2) built with Cobra v1.10.2 for command structure and Bubbletea v1.3.10 for terminal UI. The crosstown integration extends the existing `gt` command with new subcommands for ILP payment channel management and Nostr relay interaction.

**Dual Mode Operation:**
- **Interactive Mode:** `gt mayor attach` - Mayor runs continuously, processes TOON events in real-time, displays Bubbletea dashboard
- **Scriptable Mode:** `gt crosstown config`, `gt crosstown subscribe`, etc. - One-off commands for configuration and management

## Command Structure

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

## Output Formats

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

## Configuration Schema

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

## Scripting Support

**Script-Friendly Design:**

All `gt crosstown` commands support:
- **Exit codes** (0 = success, non-zero = error)
- **Structured output** (`--format json` for parsing)
- **Non-interactive mode** (`--yes` flag to skip confirmations)
- **Environment variable configuration** (no interactive prompts when vars set)

**Example Automation Script:**

```bash
#!/bin/bash