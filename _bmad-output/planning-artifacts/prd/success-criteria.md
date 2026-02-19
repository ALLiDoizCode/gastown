# Success Criteria

## User Success

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

## Business Success

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

## Technical Success

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

## Measurable Outcomes

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
