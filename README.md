# Agent skills by nandafirmans

Agent skills for frontend work: code structure, ticket drafting and conflict resolution. They work with Claude Code, Cursor, Codex, Gemini CLI and other agents that support `SKILL.md`.

## Install

Install all skills:

```bash
npx skills add nandafirmans/skills
```

Install specific skills:

```bash
npx skills add nandafirmans/skills --skill fe-structure
npx skills add nandafirmans/skills --skill ticket-draft resolve-conflicts
```

Useful options: `-g` installs globally instead of in the current project, `-a claude-code` targets a specific agent, and `-l` lists the skills without installing.

To update, run the install command again.

## Skills

| Skill | What it does |
|---|---|
| [fe-structure](skills/fe-structure/SKILL.md) | Structure JS/TS frontend code: domain/feature folders with slices, a layered data layer, small query/mutation hooks with key factories, and small global stores. |
| [ticket-draft](skills/ticket-draft/SKILL.md) | Draft a frontend ticket from findings, a bug report, a feature request, a screenshot or an API spec, without inventing requirements. |
| [resolve-conflicts](skills/resolve-conflicts/SKILL.md) | Resolve an in-progress merge, rebase or cherry-pick conflict hunk by hunk, lint the changed files, stage them and stop for approval. |

## fe-structure

For building or refactoring features in any frontend app. The reference code uses React, TanStack Query, axios and Zustand, and the skill maps each layer to Vue, Svelte and other stacks.

- **Folders:** `/<domain>/<feature>/` with `index.js` as the public API, `components/` with optional slices, and feature-level `hooks/`, `helpers/` and `context/` only when shared.
- **Placement:** code lives at the lowest level that covers every consumer and moves up only when a new consumer appears.
- **Imports:** other features import only through `index.js`, enforced with a lint rule; no barrel files elsewhere.
- **Data layer:** page → feature query/mutation hooks → generic hooks → request core → HTTP client.
- **Query keys:** one key factory per feature, generic to specific; mutations invalidate by key.
- **State:** server data in the query cache, global client state in small stores, everything else local.
- **Framework touch points:** router, env vars and navigation each sit behind one adapter.
- **Translation:** follows whatever the project already uses.

## ticket-draft

Use it when asked to "create a ticket" or "draft a ticket". The draft is returned in chat; publishing to a tracker needs your explicit approval.

- **Default format:** Title → Description → Scope of Work → Acceptance Criteria.
- **Simple changes:** Title → Description → Acceptance Criteria.
- **Notes / Out of Scope:** added only for exclusions, backend dependencies or items that still need verification.
- **Titles:** `[FE]` plus a tag for the current month and year, e.g. `[FE][NOV 26]` for November 2026.
- **Rules:** frontend scope only, backend work listed as dependencies, testable acceptance criteria, and no invented endpoints or requirements.

## resolve-conflicts

Use it during an in-progress merge, rebase or cherry-pick.

- **Sides:** explains `ours`/`theirs` for each operation, including the swap during a rebase.
- **Resolution:** keeps both sides' intent where possible and stops to ask on incompatible hunks instead of guessing. It never aborts.
- **Special files:** regenerates lockfiles and generated files instead of hand-merging; handles binary and delete-vs-modify conflicts.
- **Checks:** confirms no conflict markers remain and lints only the changed files.
- **Stops before finishing:** stages the resolved files, reports what it chose per hunk, and gives you the command to continue.

## License

MIT
