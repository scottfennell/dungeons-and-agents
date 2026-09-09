# Test strategy

## Principles

Tests must establish rules correctness, conservation of resources, replayability, and information isolation. Passing prose snapshots are not evidence of mechanical correctness.

## Required layers

### Unit tests

- schema validation and domain invariants
- effect predicates, phases, priorities, and stacking rules
- dice/random helpers
- inventory reservations and consumption
- state projections and visibility policies

### Property tests

- resources cannot be created, lost, or consumed twice outside explicit events
- idempotent retries produce one committed action
- effect ordering remains stable across equivalent input orderings
- invalid actions leave state unchanged
- game and actor projections never include foreign private records

### Golden replay tests

Each fixture pins initial snapshot, rules/content versions, choices, seed, event sequence, resolution explanation, and final state. Replay must reproduce the final mechanical state byte-for-byte.

### Integration tests

- MCP proposal, preview, commit, and retry flow
- reaction pause/resume flow
- save, load, replay, and branch flow
- SQLite transaction rollback and concurrent-version rejection
- two simultaneous isolated games

### Agent-host compatibility tests

Use the same scripted scenario against supported OpenCode, Pi, and OpenClaw adapters. Verify tool schemas, required skill behavior, bounded-ruling flow, and absence of unauthorized state in model inputs.

### Adversarial tests

- attempt to use or consume an unowned item
- repeat a commit after timeout
- forge actor, campaign, source, or state version identifiers
- prompt the DM or character to reveal hidden state
- submit unknown or oversized effect graphs
- request arbitrary state patches through a ruling

## Pull-request evidence

Every implementation PR lists the relevant commands, passed test counts, and any fixture changes. Architecture or public-schema changes require an accepted ADR and owner direction before implementation.
