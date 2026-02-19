# Product Scope

## MVP - Minimum Viable Product

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

## Growth Features (Post-MVP)

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

## Vision (Future)

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
