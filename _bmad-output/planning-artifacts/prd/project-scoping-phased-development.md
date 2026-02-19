# Project Scoping & Phased Development

## MVP Strategy & Philosophy

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

## MVP Feature Set (Phase 1)

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

## Post-MVP Features

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

## Risk Mitigation Strategy

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
