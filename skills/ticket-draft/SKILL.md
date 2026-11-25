---
name: ticket-draft
description: Draft a frontend ticket in chat from findings, a bug report, or a feature request using the user's four-section template.
---

# Frontend ticket draft

Use the evidence already available. Distinguish confirmed findings from items needing verification; do not invent endpoints, requirements, or affected screens. Return the draft in chat. Publishing to SunFish or any other tracker is a separate action and requires explicit authorization.

## Template

```markdown
## Title
[FE] <Action verb> <feature/change> in <module/page>

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
```

Keep all four headings in order. Use plain bullets for acceptance criteria, never checkboxes. Include only applicable scope bullets and make each criterion observable.
