# Dungeons and Agents — Product Requirements Document

**Status:** Approved (v1.0)  
**Date:** 2026-09-09  
**Owner:** Scott  
**Product name:** Dungeons and Agents

## 1. Product summary

Build a tabletop fantasy RPG in which one human player joins a party of AI-controlled characters led by an AI Dungeon Master. The experience should preserve the freedom and storytelling of a human-run tabletop game while making mechanical outcomes, inventory, costs, dice, and effect stacking auditable and consistent.

The initial product is a text-first, single-player campaign played through an agent host such as OpenCode, Pi, or OpenClaw. A future 3D experience agent and human multiplayer are replaceable layers over the same authoritative game protocol.

This begins as a personal, self-hosted prototype built on commercially safe foundations. Its architecture must allow multiple isolated games later without requiring the first release to operate a multi-tenant service.

## 2. Product thesis

The game should use two deliberately separate systems:

1. **An authoritative MCP game server** owns canonical state, validates actions, resolves dice and rules, consumes resources, orders effects, and records an event log.
2. **Agent skills** teach the DM and character agents how to play their roles, use the game tools, protect hidden information, and narrate resolved events.

The LLM may propose an action or make a bounded DM ruling. It must not directly mutate canonical state. Flavorful narration can be nondeterministic; mechanical resolution must be structured, validated, and replayable.

### Why this split

- A skill is prompt-level behavior, not a secure transaction boundary.
- MCP tools can enforce ownership, inventory, turn order, resource use, and valid state transitions.
- One skill per action would be hard to version, test, discover, and compose.
- Data-defined actions and effects can be added as content without adding arbitrary executable code.

## 3. Goals

### MVP goals

- Complete a 20–40 minute fantasy encounter with one human, 2–3 AI party members, and one AI DM.
- Allow freeform player language while resolving supported actions through typed game intents.
- Support combat, ability checks, dialogue, exploration, inventory, consumables, rest, and simple quests.
- Make attacks, defenses, buffs, debuffs, equipment, reactions, and consumables composable through a deterministic effect pipeline.
- Prevent impossible mutations such as using an absent potion, spending the same charge twice, acting out of turn, or silently changing a roll.
- Give each agent only the state and memories its character is allowed to know.
- Persist, pause, resume, inspect, and replay a campaign.
- Let the DM improvise without bypassing the rules engine.

### Long-term goals

- Human multiplayer with character-mediated communication and safety controls.
- Multiple settings and rule/content packs over a setting-neutral engine.
- A 3D virtual table with agents represented as seated characters, maps, minis, dice, and spatial scenes.
- Optional live-rendered or arena-style presentation of resolved action sequences.
- Creator-authored adventures, items, creatures, and effect definitions.

## 4. Non-goals for the first release

- Full implementation of every D&D rule, class, spell, monster, or setting.
- Forgotten Realms or other proprietary setting content not present in the licensed SRD.
- Photorealistic 3D, VR, real-time action combat, or motion capture.
- Open-ended execution of LLM-authored code.
- Human multiplayer, matchmaking, voice chat, real-money economies, or user-generated public content.
- Perfect simulation of a human DM.

## 5. Intellectual-property boundary

The product may use and adapt material actually included in D&D SRD 5.2.1 under CC BY 4.0 with proper attribution, license link, and change notice. It should not assume that “D&D-compatible” grants rights to Forgotten Realms, named characters, excluded monsters, trade dress, logos, or other brand identity.

Recommended launch posture:

- Use SRD 5.2.1 mechanics as the rules baseline.
- Keep the engine setting-neutral and ship a small original demo setting/content pack.
- Maintain a machine-readable provenance field for every imported rules/content record.
- Keep license and attribution output generated from the content manifest.
- Obtain legal review before commercial launch or use of D&D trademarks in marketing.

Official references:

- [D&D SRD page](https://www.dndbeyond.com/srd)
- [Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/)

## 6. Target user and initial experience

### Primary user

A solo player who wants the social and narrative feeling of a tabletop campaign without scheduling a human group. The initial owner/user is Scott; the prototype should nevertheless avoid design choices that would prevent later commercial or multi-game use.

### Session fantasy

The player sits down at a virtual table. The DM recaps the campaign and sets the scene. AI party members speak and act only from their own character knowledge and motivations. The player describes intent naturally. The system converts that intent to a legal action, previews material costs when appropriate, resolves it, and the DM narrates the result. The game can pause and resume without losing mechanical or story continuity.

## 7. Product principles

1. **Fiction is flexible; state is strict.** Narration may vary, committed mechanics may not.
2. **Intent is not execution.** Agents propose; the engine validates and commits.
3. **Every mutation has provenance.** State changes link to an action, actor, source, rules version, roll, and event.
4. **Resources are real.** Items, charges, spell slots, reactions, and currency must be owned, reserved, and consumed atomically.
5. **Knowledge is scoped.** Character agents receive projections of state, not the omniscient world state.
6. **The DM has bounded discretion.** Rulings select from constrained outcomes or create explicit, logged exceptions.
7. **Narration follows resolution.** The DM describes committed events rather than deciding mechanics in prose after the fact.
8. **Replay is a first-class feature.** A session can be reconstructed from an initial snapshot, content/rules versions, random seed, and event log.
9. **Safety is a system feature.** Character-only communication does not replace consent tools, moderation, blocking, reporting, or out-of-character safety controls.

## 8. Core gameplay loop

1. Engine emits the current scene and an actor-specific view.
2. Human or character agent states an in-fiction intention.
3. Player/character adapter proposes a typed `ActionIntent`.
4. Engine validates timing, targets, visibility, ownership, resources, and prerequisites.
5. Engine returns a preview, a legal clarification request, or a rejection with alternatives.
6. On confirmation, engine reserves costs and commits the action transaction.
7. Resolution pipeline applies rolls, modifiers, defenses, damage/healing, conditions, reactions, triggers, and costs in a stable order.
8. Engine appends canonical events and returns a `ResolutionEnvelope`.
9. DM narrates only the information visible to the participants.
10. Agents update private character memory from their authorized observations.

Exploration and dialogue use the same intent/validation/commit model, with looser timing than combat.

## 9. Recommended system architecture

### 9.1 Components

#### A. Authoritative game engine and MCP server

- Campaign/session lifecycle
- State persistence and event log
- Rules and content-pack registry
- Action validation and resolution
- Seeded dice/randomness
- Inventory and resource ledger
- Effect pipeline and reaction windows
- Per-actor state projections
- DM ruling requests and audit records
- Save, load, replay, branch, and debug tools

#### B. DM agent + DM skill

- Establish scenes, pace, tone, NPC portrayal, and consequences
- Convert unsupported freeform situations into bounded ruling requests
- Choose from engine-provided legal options
- Narrate committed events without inventing mechanical changes
- Protect secrets and disclose only authorized observations
- Apply campaign safety profile and content limits

#### C. Character agents + player-character skill

- Maintain personality, goals, bonds, flaws, and private memories
- Receive only that character's state projection
- Speak and choose actions in character
- Submit typed intents via MCP
- Never read DM-only state or other characters' private thoughts

#### D. Session director/orchestrator

- Schedules DM and character turns
- Delivers messages and state projections
- Enforces timeouts, retries, and context budgets
- Keeps out-of-character system traffic separate from in-character dialogue
- Allows a human to pause, correct, rewind, or take control of a character

#### E. Experience hosts and clients

- MVP: a chat session hosted by OpenCode, Pi, or OpenClaw using the same skills and MCP tools
- Later: compact character/combat dashboard, 2D map, and 3D experience agent
- A host may select models, invoke skills, orchestrate agents, interpret game state, and construct a presentation world
- Hosts and clients never calculate or write authoritative outcomes

### 9.2 Trust boundary

Only the engine writes canonical game state. The DM, character agents, agent hosts, human clients, and 3D experience agent are untrusted callers from the engine's perspective.

## 10. Action and effect model

### 10.1 Do not execute arbitrary agent-written code

Use a small, versioned effect algebra represented as typed JSON (or a compact DSL that compiles to it). Agents may assemble permitted nodes, but the server validates the resulting action graph against schemas, permissions, timing, and budgets.

Example conceptual nodes:

- `RequireOwnership(itemInstanceId, quantity)`
- `ReserveResource(resourceRef, amount)`
- `Roll(d20, advantageMode)`
- `AddModifier(value, sourceRef)`
- `Compare(total, defenseRef)`
- `RollDamage(dice, damageType)`
- `ModifyDamage(mode, value, sourceRef)`
- `ApplyDamage(targetId, amount, damageType)`
- `ApplyCondition(conditionId, duration, sourceRef)`
- `ConsumeReserved(resourceRef)`
- `EmitEvent(eventType, visibility)`

Fixed server functions should operate on IDs and domain types, not ambiguous integers. For example, avoid `enhancePotion(item: Int, potionLevel: Int)`. Prefer an intent such as:

```json
{
  "type": "use_item",
  "actorId": "actor_123",
  "itemInstanceId": "item_987",
  "targets": ["actor_456"],
  "mode": "enhance",
  "parameters": { "tier": 2 },
  "expectedStateVersion": 418,
  "idempotencyKey": "turn-29-action-1"
}
```

The referenced item definition supplies its legal effects. The item instance supplies ownership, quantity, charges, condition, and provenance. The engine—not the LLM—determines whether the operation is available and what it consumes.

### 10.2 Resolution phases

Use explicit phases so effects can stack without unpredictable function nesting:

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

Every effect declares:

- source and owner
- phase and priority
- scope and valid targets
- stacking group and stacking rule
- duration or remaining uses
- optional predicate
- visibility
- deterministic payload

Stable ordering should be defined by phase, priority, source timestamp, and unique ID. No effect relies on tool-call arrival order.

### 10.3 Inventory transaction rules

- Item definitions and item instances are separate.
- Inventory changes use a ledger rather than directly editing counts.
- Consumables are reserved before resolution and consumed only on commit.
- Failed validation consumes nothing.
- Interrupted or canceled actions follow an explicit refund policy.
- Commit requires an expected state version and idempotency key.
- Duplicate submissions return the original result instead of executing twice.
- Every consumption event names the item instance and the action that consumed it.

### 10.4 DM discretion

Three possible rules modes should be supported eventually:

- **Strict:** only encoded actions and effects are legal.
- **Hybrid (recommended default):** encoded mechanics plus bounded DM rulings.
- **Story:** DM may use broader narrative outcomes, still recorded as explicit ruling events.

A bounded ruling contains: question, legal parameter range or outcome choices, DM selection, rationale, visibility, and mechanical events produced. It cannot directly patch arbitrary state.

## 11. Canonical data model

Minimum top-level records:

- `Campaign`: identity, setting pack, rules version, safety profile, participants
- `Session`: timeline position, active scene, status, random seed commitment
- `Scene`: location, participants, mode, turn/round state, public facts
- `Actor`: stats, resources, conditions, position, controller, visibility policy
- `CharacterKnowledge`: observations, beliefs, secrets, relationships, private memory refs
- `ItemDefinition`: immutable mechanics and content provenance
- `ItemInstance`: owner/location, quantity, charges, state, unique properties
- `ActionDefinition`: schema, prerequisites, costs, effect graph, rules provenance
- `ActionIntent`: proposed action and caller-provided choices
- `EffectInstance`: source, phase, stack rule, duration, payload
- `Event`: immutable state transition or observation
- `Ruling`: bounded DM adjudication and rationale
- `StateSnapshot`: replay/checkpoint material

Use append-only events plus periodic snapshots. Derived current state can be rebuilt from the log.

## 12. Initial MCP surface

The exact names may change, but the server should expose a small capability-oriented API:

- `campaign.create`, `campaign.load`, `campaign.pause`
- `view.get_self`, `view.get_scene`, `view.get_legal_actions`
- `action.propose`, `action.preview`, `action.commit`
- `reaction.list`, `reaction.respond`
- `ruling.request`, `ruling.resolve`
- `inventory.inspect`, `character.inspect`
- `session.events`, `session.recap`
- `debug.replay` and `debug.explain_resolution` (development/admin only)

Avoid one MCP tool per spell, item, or move. Those are content records executed by the stable action/effect engine.

## 13. Agent communication and multiplayer safety

### Initial single-player isolation

- DM receives full world state and private scenario data.
- Each character agent receives its own sheet, private memory, and observable scene projection.
- Agents do not share transcripts, chain-of-thought, tool contexts, or private state.
- In-character messages pass through the session director and become observable events.
- Private DM-to-character messages are explicit scoped events.

### Later human multiplayer

Character mediation can reduce direct abuse, but it is not sufficient by itself. Required controls include:

- table code of conduct and campaign content profile
- lines, veils, pause/X-card, and immediate out-of-character safety channel
- mute, block, kick, report, and host controls
- rate limiting and harassment/spam classifiers
- clear identity and privacy policy
- separation of character conflict from player targeting
- audit trail with appropriate retention and access controls

The system should never force an abused player to remain “in character” to ask for help.

## 14. 3D experience-agent path

The future 3D component is an **experience agent and host**, not merely a renderer. It may use skills, MCP tools, and LLMs to interpret game state, construct a 3D world, choose staging, and direct presentation. It remains outside the game's authority boundary: mechanics, content identities, inventory, and canonical state still live in the MCP game server.

This boundary permits radically different experiences over the same game—a coding-agent chat, traditional virtual table, explorable 3D world, or arena-style presentation—without forking the rules engine.

### Progressive delivery

1. Text interface through OpenCode, Pi, or OpenClaw
2. 2D encounter map driven by scene positions
3. 3D experience agent with seats, minis, dice, maps, avatars, and generated scene staging
4. Cinematic action visualization derived from resolution envelopes
5. Optional arena/live-action presentation mode

The engine must expose semantic events such as `actor.moved`, `attack.resolved`, `damage.applied`, and `condition.added`, plus stable scene/entity identifiers and presentation-safe descriptions. Experience agents map those events to scenes and animations. This keeps presentation swappable and avoids coupling game logic to Unity, Godot, Unreal, or web 3D during MVP.

## 15. Functional requirements

### P0 — vertical slice

- Create/load one campaign and one encounter
- Human player plus at least two isolated character agents and one DM agent
- Chat play through at least one supported agent host; OpenCode, Pi, and OpenClaw are the portability targets
- Freeform intent conversion for a constrained action catalog
- Initiative and turn state
- Attacks, defenses, damage, healing, conditions, and reactions
- Inventory ownership and one consumable item flow
- Seeded rolls and complete resolution trace
- Per-character views and private information test
- Save/resume and deterministic replay
- DM narration from committed events
- Human override, pause, and rollback to a checkpoint

### P1 — solo campaign

- Exploration/dialogue time
- Quests, travel, rests, advancement, shops, loot, spell/resource recovery
- Campaign recap and long-term character memory
- Multiple scenes and encounters
- Adventure/content pack loader
- Safety-profile configuration
- Text interaction usable without inspecting raw JSON
- Compatibility test suite for host-neutral MCP and skill behavior

### P2 — presentation and community

- 2D/3D client
- Human multiplayer and moderation controls
- Creator tools and content validation
- Spectator/replay mode
- Additional settings and rules packs

## 16. Quality attributes

- **Determinism:** given the same snapshot, rules/content versions, committed choices, and seed, mechanics replay identically.
- **Auditability:** any modifier or state change can be explained with ordered sources.
- **Security:** callers cannot access unauthorized projections or mutate state outside valid tools.
- **Resilience:** retries do not duplicate actions or resource consumption.
- **Latency:** a normal non-LLM engine action resolves in under 250 ms locally; complete narrated turns target under 8 seconds in MVP.
- **Portability:** headless engine, MCP surface, and skill contract do not depend on a specific model, agent host, or visual client.
- **Game isolation:** campaign IDs, state, events, secrets, and agent views cannot cross game boundaries.
- **Deployment:** the first release is self-hosted; persistence and authorization seams permit later multi-game operation.
- **Versioning:** campaigns pin rule and content versions; migrations are explicit.
- **Testability:** golden replay tests cover the resolution engine; property tests cover conservation and idempotency.

## 17. Success criteria for the first vertical slice

1. A new player completes the encounter without reading tool schemas.
2. No agent can use an item it does not own or consume one twice under retries.
3. Three simultaneous attack/defense modifiers resolve in a documented stable order.
4. A reaction can interrupt the pipeline without corrupting state.
5. Hidden information is absent from unauthorized agent inputs and outputs.
6. Replaying the event log reproduces the final mechanical state byte-for-byte.
7. The DM's narration does not contradict the resolution envelope in the test corpus.
8. The campaign can stop and resume from a saved checkpoint.

## 18. Recommended implementation sequence

### M0 — architecture spike

- Choose language/runtime and persistence.
- Establish a TypeScript workspace and a storage interface backed initially by SQLite.
- Specify schemas for state, intents, effects, events, projections, and rulings.
- Implement one attack with two modifiers, one reaction, and one consumable.
- Prove idempotent commits and deterministic replay.
- Run with scripted callers before adding agents.

**Exit:** a headless test replays identically and explains every modifier and resource mutation.

### M1 — playable encounter

- Add MCP transport and tools.
- Add DM and character skills.
- Add session orchestration and view isolation.
- Add a small original, disposable adventure/content pack and character party.
- Prove chat play through one agent host and document the portable setup for OpenCode, Pi, and OpenClaw.

**Exit:** one human completes a 20–40 minute encounter with AI companions and AI DM.

### M2 — persistent solo campaign

- Add exploration, dialogue, quests, rest, advancement, memory, recap, and content packs.
- Improve failure recovery, observability, and authoring tools.

**Exit:** a campaign supports multiple resumable sessions.

### M3 — visual table prototype

- Emit stable presentation events.
- Build a 2D or simple 3D client as a replaceable consumer.

**Exit:** the visual state remains synchronized through reconnect and replay.

### M4 — multiplayer safety prototype

- Add human seats, consent, moderation, privacy, and abuse controls.
- Test with invited groups before any open matchmaking.

**Exit:** controlled playtests demonstrate usable safety and moderation flows.

## 19. Project steward agent plan

Create a dedicated persistent **Project Steward** agent after the remaining launch gates in Section 21 are settled. It manages the project record; bounded coding tasks go to disposable implementation workers.

### Steward responsibilities

- Treat the PRD, architecture decision records, GitHub issues, and pull requests as the durable source of truth.
- Convert accepted milestones into small issues with testable acceptance criteria.
- Triage only explicitly labeled intake work.
- Detect duplicate branches, pull requests, or active workers before delegation.
- Gate changes to architecture, data model, public API, licensing, privacy, security, dependencies, deployment, or user-visible direction for Scott's decision.
- Delegate bounded implementation in isolated worktrees.
- Require tests, conventional commits, issue-linked draft PRs, and compact evidence.
- Report ready-for-review work, blockers, decisions, skipped work, and next check.
- Never merge, release, change production, rotate secrets, alter permissions, incur cost, or perform destructive Git actions unless the exact action is explicitly authorized.

### Required stewardship contract

- Repository: `scottfennell/dungeons-and-agents` (`git@github.com:scottfennell/dungeons-and-agents.git`)
- Default branch: `main`
- Intake label: proposed `steward:ready`
- Direction label: proposed `decision:needed`
- Concurrency: proposed 1 implementation issue at a time until M1
- Allowed GitHub writes: issues, labels, comments, branches, draft PRs, and CI runs for authorized intake issues
- Prohibited by default: merge, release, production/deployment, secrets, billing, permissions, destructive Git
- Status destination and cadence: current direct conversation after meaningful progress or when a decision is needed; scheduled cadence TBD
- Test commands and definition of done: recorded in repository instructions

### Agent bootstrap sequence

1. Confirm the remaining launch gates: human party-control model, product name, status cadence, and content-safety defaults.
2. Create repository skeleton with this PRD, `ARCHITECTURE.md`, ADR directory, contribution rules, and test strategy.
3. Create labels, milestones, issue templates, and the M0 epic.
4. Configure a dedicated persistent OpenClaw agent with the Project Steward skill and only the credentials/actions in the stewardship contract.
5. Give the agent the repository URL, default branch, labels, concurrency, status route, and prohibited actions.
6. Run a read-only dry run; verify that it classifies work and requests decisions without implementing direction-gated issues.
7. Authorize one bounded spike issue; verify isolated implementation, tests, draft PR, and report.
8. Expand autonomy only after the workflow is reliable.

### Suggested first issues

1. Define domain schema and invariants.
2. Specify effect algebra and phase ordering.
3. Implement event store, snapshots, idempotency, and deterministic RNG.
4. Implement attack/defense/reaction vertical slice.
5. Implement item instance, reservation, and potion consumption.
6. Implement actor-specific projections and leakage tests.
7. Expose the vertical slice through MCP.
8. Build scripted encounter harness and golden replay tests.
9. Draft DM/player-character skills against the proven MCP surface.
10. Run the first agent-driven encounter and capture failure cases.

## 20. Major risks and mitigations

| Risk | Mitigation |
|---|---|
| LLM invents mechanics or inventory | Engine-authoritative commits; narration consumes resolution envelopes |
| Stackable effects become a general-purpose programming language | Small closed effect algebra, schema validation, phase budgets, no arbitrary execution |
| DM improvisation is strangled | Hybrid mode with bounded, logged rulings and explicit homebrew content workflow |
| Hidden state leaks between agents | Server-generated projections, separate contexts, adversarial leakage tests |
| Agent turn latency/cost explodes | Small state views, compact events, deterministic engine work, model tiers, turn timeouts |
| Rules/content version drift breaks campaigns | Pin versions per campaign; explicit migrations; replay fixtures |
| D&D setting IP creates launch risk | SRD-only mechanics, original world, provenance manifest, legal review |
| “Character-only” communication hides harassment | Separate safety channel plus moderation, controls, and audit policy |
| 3D scope consumes the project | Headless vertical slice first; presentation only consumes semantic events |
| Dedicated agent drifts or overreaches | Written stewardship contract, decision gates, issue labels, concurrency cap, no merge/release by default |

## 21. Product decision record

### Confirmed 2026-09-09

- **Product posture:** personal prototype built on commercially safe foundations.
- **Rules:** SRD 5.2.1/5.5e with required attribution.
- **Setting:** setting-neutral engine with a disposable original demo setting/content pack.
- **Companions:** independent personalities with goals, loyalties, and reasonable disagreement.
- **DM authority:** hybrid—encoded mechanics plus bounded, logged rulings.
- **Death/rewind:** death matters; checkpoint rewinds create explicit branched timelines rather than rewriting history.
- **Vertical slice:** short dungeon containing dialogue, exploration, a trap, and combat.
- **3D boundary:** a future experience agent may use skills, MCP, and LLMs to construct a 3D world, but the game remains portable and authoritative outside that agent.
- **Initial interface:** chat through OpenCode, Pi, or OpenClaw.
- **Human multiplayer communication:** character-first, with out-of-character safety and logistics available.
- **Implementation:** TypeScript.
- **Deployment:** self-hosted first, with an eventual path to multiple isolated games.
- **Steward autonomy:** may create and triage issues, delegate bounded work, run tests/CI, and open draft PRs; Scott approves architecture and merges.
- **Repository:** `https://github.com/scottfennell/dungeons-and-agents`, public, default branch `main`; access verified 2026-09-09.

- **Human control:** the player controls one character and may make nonbinding tactical requests to companions.
- **Tone/content:** campaign profiles are configurable; the demo defaults to heroic PG-13 fantasy.
- **Live homebrew:** the DM may make temporary bounded rulings; persistent or reusable mechanics require campaign-owner approval.
- **Effect authoring:** developers may edit validated JSON/DSL directly; ordinary players use an agent-assisted validated content builder.
- **Steward reporting:** report material results, blockers, and decisions; no scheduled report initially.
- **Repository visibility:** public.

### Decisions safely deferred beyond M0

- Public/open-source/commercial distribution decision after the personal prototype
- Anonymous or open matchmaking policy
- Romance, PvP, betrayal, deception, and secret-goal opt-ins
- Private player-to-DM signaling details
- 3D runtime choice: web 3D, Godot, Unity, Unreal, or another host
- Multi-game deployment shape: one process per game, multi-campaign server, or hosted multi-tenant service
- Model selection and operating budget

## 22. Fixed technical direction

- **Action representation:** typed JSON effect algebra; optional human-friendly DSL later.
- **Authority:** MCP server owns state; skills own role behavior and narration policy.
- **Host boundary:** OpenCode, Pi, and OpenClaw should be interchangeable chat hosts over the same MCP and skill contracts.
- **Persistence:** append-only SQLite event store plus snapshots behind a storage interface; preserve a path to PostgreSQL or another shared store later.
- **Isolation:** every state read/write is scoped to a game/campaign and actor view; no global campaign singleton.
- **Agent topology:** persistent steward, one DM, isolated character agents, session director, disposable coding workers.
- **3D:** no 3D implementation before the chat vertical slice; design semantic event output so a future experience agent can consume it.

## 23. Approval and next gate

Scott approved the v1 product defaults on 2026-09-09. The next gate is operational: bootstrap the repository, activate Questwright under its stewardship contract, approve the M0 issue breakdown, and prove one issue-to-draft-PR execution cycle.
