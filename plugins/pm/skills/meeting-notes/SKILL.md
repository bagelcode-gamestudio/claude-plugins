---
description: Meeting note template and conventions for structured team communication
globs:
  - "**/*.md"
  - "**/*.txt"
---

# Meeting Notes Template

When creating or organizing meeting notes, follow this structure:

## Required Sections
- **Date**: YYYY-MM-DD format
- **Attendees**: List of participants
- **Agenda**: Numbered items discussed
- **Decisions**: Clearly stated outcomes with owners
- **Action Items**: `[ ] Task — @owner — due YYYY-MM-DD`

## Conventions
- File naming: `{YYYY-MM-DD}_{meeting-type}_{topic}.md`
- Examples:
  - `2026-03-12_standup_sprint-42.md`
  - `2026-03-12_review_combat-system.md`
- One file per meeting, stored in `/docs/meetings/`

## Action Item Rules
- Every action item must have an owner and due date
- Format: `[ ] {description} — @{owner} — due {YYYY-MM-DD}`
- Review open action items at start of next meeting
- Completed items: `[x] {description} — @{owner} — done {YYYY-MM-DD}`

## Distribution
- Share within 24 hours of meeting
- Tag relevant people who were absent
- Link to related tickets or documents