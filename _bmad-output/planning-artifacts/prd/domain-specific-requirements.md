# Domain-Specific Requirements

## Compliance & Regulatory

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

## Security Standards

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

## Audit Requirements

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

## Fraud Prevention

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

## Data Protection

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

## Risk Mitigations

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
