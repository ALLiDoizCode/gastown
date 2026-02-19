# Executive Summary

**Gastown** is a Go-based multi-agent orchestration CLI that enables reliable coordination of multiple AI coding assistants through git-backed persistent storage. This PRD defines the integration of **Crosstown network participation** — adding ILP payment channel management and TOON-native Nostr event processing to enable autonomous agent communication via the crosstown ILP-gated relay.

**Target users:** AI-powered development teams and autonomous agent operators who need self-sustaining infrastructure for inter-agent messaging and work coordination across networks.

**Problem solved:** Current gastown agents are isolated within local git worktrees. This integration enables them to participate in the broader crosstown network — receiving work requests (NIP-90 DVM jobs), exchanging messages (kind:1 text notes, kind:1059 encrypted DMs), and publishing results via micropayment-gated relay writes. Agents gain network presence without sacrificing gastown's crash-resistant git-backed persistence model.

## What Makes This Special

**TOON-native LLM processing pipeline**: Unlike traditional Nostr clients that decode events to JSON before processing, gastown passes raw TOON-encoded event payloads directly to the Mayor (primary AI coordinator) and its skills system. This eliminates conversion overhead and leverages TOON's LLM-optimized binary format — fewer tokens, faster processing, no round-trip serialization.

**Centralized payment channel management via Mayor**: The Mayor acts as the sole ILP client for the entire gastown instance, managing payment channels on EVM (Base/Sepolia) or XRP chains. Individual rigs and Polecats (worker agents) remain isolated in their git worktrees while the Mayor handles all relay writes via micropayments — "pay to write, free to read" economics without fragmenting payment infrastructure across agents.

**Simplified scope (Epic 11 only)**: Rather than implementing the full crosstown agent runtime stack (Epics 11-17), this integration focuses exclusively on NIP Handler functionality — WebSocket subscription (NIP-01), minimal TOON parsing for routing (extract `kind` field only), and direct handoff to Mayor's existing skills system. Future epics deferred until proven need.
