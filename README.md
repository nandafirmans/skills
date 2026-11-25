# Agent skills by nandafirmans

Agent skills for structuring frontend code, usable with Claude Code, Cursor, Codex and other agents that support `SKILL.md`.

## Install

```bash
npx skills add nandafirmans/skills
```

Install only one skill:

```bash
npx skills add nandafirmans/skills --skill fe-structure
```

## Skills

| Skill | What it does |
|---|---|
| [fe-structure](skills/fe-structure/SKILL.md) | Structure JS/TS frontend code: domain/feature folders with slices, a layered data layer, small query/mutation hooks with key factories, and small global stores. Works with React, Vue, Svelte and other frameworks. |

## fe-structure at a glance

- **Folders:** `/<domain>/<feature>/` with `index.js` as the public API, `components/` with optional slices, and feature-level `hooks/`, `helpers/` and `context/` only when shared.
- **Placement:** code lives at the lowest level that covers every consumer and moves up only when a new consumer appears.
- **Data layer:** page → feature query/mutation hooks → generic hooks → request core → HTTP client.
- **Query keys:** one key factory per feature, generic to specific; mutations invalidate by key.
- **State:** server data in the query cache, global client state in small stores, everything else local.
- **Framework-agnostic:** router, env vars and navigation each sit behind one adapter.

## License

MIT
