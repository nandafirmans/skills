---
name: ticket-draft
description: Draft a frontend engineering ticket in chat from findings, a bug report, a feature request, a screenshot or an API spec. Use when asked to "create a ticket", "draft a ticket" or write one for an API integration.
---

# Frontend ticket draft

Use the evidence already available: the conversation, code, screenshots, API specs or bug reports. Do not invent endpoints, fields, requirements or affected screens. Return the draft in chat. Publishing to any tracker (Jira, Linear, GitHub Issues, etc.) is a separate action and requires explicit authorization.

## Choose the format

- **Default:** Title → Description → Scope of Work → Acceptance Criteria.
- **Simple change** (copy, styling, a single small fix): Title → Description → Acceptance Criteria.
- **Notes / Out of Scope:** add only when there are important exclusions, backend dependencies, or open questions that need verification.

## Template

```markdown
## Title
[FE][<MMM YY>] <Action verb> <feature/change> in <module/page>

## Description
<1–3 sentences: what needs to change, where, and why.>

## Scope of Work
- <UI behavior: what the user sees or does>
- <API integration, if applicable: endpoints, methods, request/response mapping>
- <Relevant states: loading, empty, error, success>
- <Validation, conditions, permissions, or visibility>
- <Compatibility: existing flows that must keep working>

## Acceptance Criteria
- <Specific, testable behavior>
- <Specific, testable behavior>

## Notes / Out of Scope
- <Backend dependency, exclusion, or item needing verification>
```

## Rules

- **Title:** action-oriented, prefixed with `[FE]` and a date tag for the current month and year: three-letter month in capitals plus the two-digit year, e.g. `[FE][NOV 26]` for November 2026. Take the date from the environment (system date or a time tool) rather than guessing; if it isn't available, ask. Drop the date tag only if the user asks.
- **Scope of Work:** include only the bullets that apply. Each one is a concrete frontend task.
- **Frontend only:** keep backend work out of scope. If the frontend depends on backend work (a missing endpoint or field), list it under Notes as a dependency, not as a task.
- **Acceptance Criteria:** plain bullets, never checkboxes. Each criterion is observable and testable, covers the relevant loading/error/validation states, and includes that existing functionality is unaffected when that's a risk.
- **Confirmed vs unverified:** state confirmed findings plainly; put anything that still needs verification under Notes as an open question.
- **Scope:** only what was requested. Keep it concise and technically accurate. Write in English unless asked otherwise.
