# User Journeys

## Journey 1: Mayor (Primary AI Agent) - CI/CD Cross-Architecture Build

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

## Journey 2: Gastown Operator (Human) - Setup and Monitoring

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

## Journey 3: Remote Mayor-Beta (AMD64 Linux Server) - CI/CD Service Provider

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

## Journey 4: Skills Developer (Human) - Extending TOON Event Handling

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

## Journey Requirements Summary

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
