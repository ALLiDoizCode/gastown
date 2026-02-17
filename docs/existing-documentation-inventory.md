# Existing Documentation Inventory

**Generated:** 2026-02-17
**Project:** Gas Town

## Root Level Documentation (7 files)

| File | Type | Description |
|------|------|-------------|
| [README.md](../README.md) | readme | Main project README - overview, installation, quick start |
| [CONTRIBUTING.md](../CONTRIBUTING.md) | contributing | Contribution guidelines |
| [AGENTS.md](../AGENTS.md) | agents | Agent-specific documentation |
| [CHANGELOG.md](../CHANGELOG.md) | changelog | Version history and release notes |
| [CODE_OF_CONDUCT.md](../CODE_OF_CONDUCT.md) | community | Community code of conduct |
| [RELEASING.md](../RELEASING.md) | process | Release process documentation |
| [SECURITY.md](../SECURITY.md) | security | Security policies and reporting |

## Core Documentation (docs/) (9 files)

| File | Type | Description |
|------|------|-------------|
| [HOOKS.md](./HOOKS.md) | architecture | Git hooks and persistent storage system |
| [INSTALLING.md](./INSTALLING.md) | setup | Installation instructions |
| [beads-native-messaging.md](./beads-native-messaging.md) | integration | Beads integration messaging protocol |
| [glossary.md](./glossary.md) | reference | Terminology and concepts glossary |
| [overview.md](./overview.md) | overview | Project overview |
| [reference.md](./reference.md) | reference | Command reference |
| [formula-resolution.md](./formula-resolution.md) | design | Formula resolution system |
| [mol-mall-design.md](./mol-mall-design.md) | design | Molecule mall design |
| [why-these-features.md](./why-these-features.md) | rationale | Design rationale |

## Concepts Documentation (docs/concepts/) (6 files)

| File | Type | Description |
|------|------|-------------|
| [convoy.md](./concepts/convoy.md) | concept | Convoy work tracking system |
| [identity.md](./concepts/identity.md) | concept | Agent identity management |
| [integration-branches.md](./concepts/integration-branches.md) | concept | Integration branch workflow |
| [molecules.md](./concepts/molecules.md) | concept | Molecule system (Beads formulas) |
| [polecat-lifecycle.md](./concepts/polecat-lifecycle.md) | concept | Worker agent lifecycle |
| [propulsion-principle.md](./concepts/propulsion-principle.md) | concept | Git-backed propulsion mechanism |

## Design Documentation (docs/design/) (17 files)

| File | Type | Description |
|------|------|-------------|
| [architecture.md](./design/architecture.md) | architecture | Overall system architecture |
| [agent-provider-interface.md](./design/agent-provider-interface.md) | design | Agent provider abstraction |
| [convoy-lifecycle.md](./design/convoy-lifecycle.md) | design | Convoy lifecycle management |
| [dog-pool-architecture.md](./design/dog-pool-architecture.md) | design | Dog pool architecture |
| [dolt-storage.md](./design/dolt-storage.md) | design | Dolt database storage |
| [escalation-system.md](./design/escalation-system.md) | design | Work escalation system |
| [escalation.md](./design/escalation.md) | design | Escalation patterns |
| [federation.md](./design/federation.md) | design | Town federation system |
| [hooks-registry-design.md](./design/hooks-registry-design.md) | design | Hooks registry architecture |
| [ledger-export-triggers.md](./design/ledger-export-triggers.md) | design | Ledger export trigger system |
| [mail-protocol.md](./design/mail-protocol.md) | design | Inter-agent mail protocol |
| [operational-state.md](./design/operational-state.md) | design | Operational state management |
| [plugin-system.md](./design/plugin-system.md) | design | Plugin system architecture |
| [property-layers.md](./design/property-layers.md) | design | Property layering system |
| [watchdog-chain.md](./design/watchdog-chain.md) | design | Watchdog monitoring chain |
| [witness-at-team-lead.md](./design/witness-at-team-lead.md) | design | Witness/team lead pattern |
| [at-spike-report.md](./design/at-spike-report.md) | spike | Architecture spike report |

## Examples (docs/examples/) (1 file)

| File | Type | Description |
|------|------|-------------|
| [hanoi-demo.md](./examples/hanoi-demo.md) | example | Tower of Hanoi demo workflow |

## Issues/Planning (docs/issues/) (2 files)

| File | Type | Description |
|------|------|-------------|
| [dashboard-workflow-improvements.md](./issues/dashboard-workflow-improvements.md) | issue | Dashboard improvement tracking |
| [fix-dolt-migrate-redirect.md](./issues/fix-dolt-migrate-redirect.md) | issue | Bug fix documentation |

## Part-Specific Documentation

### NPM Package
- [npm-package/README.md](../npm-package/README.md) - NPM distribution package README

### Internal Components
- [.beads/README.md](../.beads/README.md) - Beads integration README
- [internal/formula/README.md](../internal/formula/README.md) - Formula system README

## Documentation Summary

- **Total Files:** 42 markdown files
- **Categories:**
  - Root documentation: 7
  - Core docs: 9
  - Concept docs: 6
  - Design docs: 17
  - Examples: 1
  - Issues: 2
- **Well-documented areas:**
  - System architecture
  - Core concepts (convoy, hooks, polecats, molecules)
  - Agent lifecycle and identity
  - Integration protocols
  - Plugin and federation systems
