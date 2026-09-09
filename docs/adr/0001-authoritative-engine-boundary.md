# ADR 0001: Separate authoritative mechanics from agent hosts

- Status: Accepted
- Date: 2026-09-09
- Decider: Scott Fennell

## Context

The game needs freeform LLM narration and improvisation while enforcing inventory, costs, dice, stackable effects, hidden information, and deterministic replay. It must run through text agents initially and support a future 3D experience without moving game authority into that client.

## Decision

Canonical rules and state live in a deterministic TypeScript engine exposed through MCP. Skills define DM and character behavior. Agent hosts—including OpenCode, Pi, OpenClaw, and a future 3D experience agent—submit typed intents and consume authorized views and committed semantic events.

Only the engine may commit state. LLMs may propose intents and select bounded rulings but may not apply arbitrary patches or execute authored code.

## Consequences

- Inventory and action legality can be enforced transactionally.
- Mechanical sessions can be replayed and audited.
- Agent hosts and models remain replaceable.
- 3D presentation can evolve independently.
- The engine requires a closed effect algebra and explicit projection model.
- Improvised mechanics require a bounded-ruling or reviewed content-authoring path.
