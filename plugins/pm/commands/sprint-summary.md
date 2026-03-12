---
description: Generate a sprint summary from recent commits and tickets
argument-hint: [sprint-name]
---

# Sprint Summary

Generate a concise sprint summary from recent activity.

## Instructions

1. Gather data from:
   - Recent git commits (last 2 weeks or specified sprint period)
   - Any linked ticket references in commit messages
2. Organize by:
   - **Completed**: Features and fixes shipped
   - **In Progress**: Work started but not merged
   - **Blocked**: Items with known blockers
   - **Carried Over**: Planned but not started
3. Output format:
   - Sprint name and date range
   - Summary stats (completed / total planned)
   - Categorized list with ticket IDs where available
   - Key risks or blockers for next sprint
4. Keep it readable for non-technical stakeholders.