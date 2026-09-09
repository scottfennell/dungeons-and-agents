# Dungeons and Agents

Dungeons and Agents is a self-hosted, agent-driven tabletop RPG. A deterministic MCP server owns rules and canonical state; DM and character skills provide interpretation, role-play, and narration; interchangeable hosts such as OpenCode, Pi, OpenClaw, and future 3D agents provide the player experience.

## Status

The project is in **M0 architecture-spike planning**. No playable engine exists yet.

## Product boundary

```text
Skills + MCP + content + persisted state = portable game

Agent hosts and 3D experiences = replaceable interfaces
```

Only the game engine may mutate canonical state. LLMs propose intents and bounded rulings; the engine validates and commits them.

## Documents

- [Product requirements](docs/PRD.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Test strategy](docs/TEST_STRATEGY.md)
- [Questwright stewardship contract](docs/PROJECT_STEWARD_CONTRACT.md)
- [Architecture decisions](docs/adr/README.md)

## Licensing

Repository code is licensed under GPL-3.0; see [LICENSE](LICENSE).

Future D&D SRD-derived rules content must retain separate CC BY 4.0 attribution and provenance. The engine is setting-neutral, and the initial adventure will use an original demo setting.
