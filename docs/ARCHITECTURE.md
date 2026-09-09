# Architecture

**Status:** Accepted direction; implementation pending  
**Date:** 2026-09-09

## System boundary

Dungeons and Agents separates authoritative mechanics from agent behavior and presentation.

```text
Human
  |
Experience host (OpenCode, Pi, OpenClaw, future 3D agent)
  |
Session director -> DM agent / isolated character agents
  |
MCP game server
  |
Deterministic engine -> event store / snapshots / content registry
```

Only the engine writes canonical game state. Hosts, DM agents, character agents, and visual experiences are untrusted callers.

## Authoritative engine

The TypeScript engine owns:

- campaign and session lifecycle
- rules/content versions and provenance
- actor-specific views and hidden-information boundaries
- typed action validation
- seeded random resolution
- ordered effects and reaction windows
- inventory/resource reservations and consumption
- immutable events and periodic snapshots
- replay, branch, and resolution explanations

The engine should remain transport-independent. MCP exposes application capabilities but does not contain domain logic.

## Action model

Agents submit typed `ActionIntent` values. Action definitions compile to a closed JSON effect algebra. The engine validates node types, parameters, permissions, phases, targets, and resource budgets before execution.

Resolution phases:

1. `DECLARE`
2. `VALIDATE`
3. `RESERVE_COSTS`
4. `BUILD_CHECK_OR_ATTACK`
5. `PRE_ROLL_MODIFIERS`
6. `ROLL`
7. `POST_ROLL_MODIFIERS`
8. `DETERMINE_OUTCOME`
9. `BUILD_EFFECT_OR_DAMAGE`
10. `PRE_APPLY_MODIFIERS`
11. `REACTION_WINDOW`
12. `APPLY`
13. `TRIGGERS`
14. `CONSUME_COSTS`
15. `COMMIT`

Effect ordering is stable by phase, priority, source timestamp, and unique ID. Arrival order is never semantic.

## Transaction and replay model

- A proposal or preview does not mutate state.
- Commit supplies an expected state version and idempotency key.
- Costs are reserved before resolution and consumed atomically at commit.
- Duplicate submissions return the original resolution.
- Events are append-only and include rules/content versions and random evidence.
- Checkpoint rewinds create a new branch; they do not rewrite the prior branch.
- Replay begins from a snapshot and reproduces mechanical state from committed events.

## Information model

The server produces projections rather than exposing world state:

- the DM receives scenario-authorized omniscient state
- a character receives its sheet, private memory references, and observable scene facts
- the human receives only the views allowed by campaign/player role
- private messages are explicit scoped events

Game ID, campaign ID, actor identity, and authorization scope are mandatory at every state boundary. Tests must prove that two games and two actor views cannot leak into each other.

## Skills and bounded rulings

DM and character skills define role behavior, tool-use protocol, narration constraints, and knowledge handling. They are not authoritative rules modules.

The DM may select temporary bounded rulings within engine-provided ranges or outcomes. Persistent or reusable homebrew mechanics require campaign-owner approval and a versioned content definition.

## Persistence and deployment

M0 uses SQLite behind a storage interface. The schema must avoid process-global campaign state and support multiple isolated game IDs. A future deployment may replace SQLite with PostgreSQL or another shared store and add tenant authorization without rewriting domain mechanics.

## Host and 3D portability

OpenCode, Pi, and OpenClaw should consume the same MCP surface and skill contract. A future 3D experience agent may interpret semantic events, build scenes, and direct presentation, but cannot resolve or mutate mechanics.

The engine emits presentation-safe semantic events with stable entity IDs—for example `actor.moved`, `attack.resolved`, `damage.applied`, and `condition.added`.

## M0 architecture proof

The first headless spike must prove:

- typed state, intent, effect, event, projection, and ruling schemas
- an attack with two stackable modifiers
- a defense/reaction interrupting at a documented phase
- a potion reserved and consumed atomically
- seeded rolls and a complete resolution explanation
- optimistic version checks and idempotent retries
- deterministic replay
- isolation between games and actor views

Agent orchestration and prose quality begin after this mechanical spine passes. The 3D experience begins after the chat vertical slice.
