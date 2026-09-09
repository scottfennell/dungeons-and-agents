# Repository instructions

## Authority

Before changing this repository, read:

1. `docs/PRD.md`
2. `docs/ARCHITECTURE.md`
3. `docs/PROJECT_STEWARD_CONTRACT.md`
4. Applicable records under `docs/adr/`

The accepted PRD and ADRs define product and architecture direction. GitHub issues and pull requests are the durable work record.

## Work rules

- Implement only an issue labeled `steward:ready` or work explicitly authorized by Scott.
- Use an isolated worktree for delegated implementation.
- Keep one implementation issue active at a time through M1.
- Prefer the smallest change satisfying explicit acceptance criteria.
- Link every pull request to its issue and include test evidence.
- Use conventional commits.
- Open draft pull requests; do not merge them.
- Stop for owner direction before changing product behavior, architecture, persisted schemas, the public MCP surface, licensing, security, privacy, deployment, or material dependencies.
- Do not release, deploy, alter secrets or permissions, incur costs, force-push, or rewrite shared history.

## Architectural invariants

- Only the engine writes canonical state.
- LLMs and hosts are untrusted callers that submit typed intents.
- Actions use a closed, validated effect algebra; never execute arbitrary LLM-authored code.
- Every mutation has source, actor, action, rules/content version, roll, and event provenance.
- Inventory and other resources are reserved and consumed atomically.
- Commits use expected state versions and idempotency keys.
- Mechanics must replay deterministically from pinned versions, committed choices, seed, snapshot, and events.
- Every read and write is scoped to a game/campaign and authorized actor view.
- OpenCode, Pi, OpenClaw, and future experience agents use the same MCP and skill contracts.

## Licensing and content

- Code is GPL-3.0.
- SRD 5.2.1-derived material must be identifiable through machine-readable provenance and attributed under CC BY 4.0.
- Do not introduce Forgotten Realms, protected D&D product identity, logos, named characters, or content outside the licensed SRD.
- Demo setting material must be original and separable from the setting-neutral engine.

## Verification

Commands will be fixed when the TypeScript workspace is created. Until then, documentation changes must pass `git diff --check` and preserve internal links.
