---
description: Quick review of the current file or staged changes
argument-hint: [file-path]
---

# Quick Review

Review code and give brief, actionable feedback.

## Instructions

1. If a file path is given, review that file. Otherwise, review staged git changes (`git diff --cached`).
2. Check for:
   - Obvious bugs or logic errors
   - Missing error handling
   - Naming clarity
3. Output format:
   - 🟢 Good: things done well (1-2 items)
   - 🟡 Suggest: improvements (1-3 items)
   - 🔴 Fix: must-fix issues (if any)
4. Keep it short — max 10 lines total.
