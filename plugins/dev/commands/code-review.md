---
description: Review code for bugs, conventions, and improvements
argument-hint: [file-path]
---

# Code Review

Perform a thorough code review on the given file or staged changes.

## Instructions

1. If a file path is given, review that file. Otherwise, review staged git changes (`git diff --cached`).
2. Check for:
   - Logic errors and potential bugs
   - Unity/C# or Node.js/TS anti-patterns
   - Missing error handling or null checks
   - Naming and convention violations
   - Performance concerns
3. Output format:
   - **Pass**: Things done well (1-2 items)
   - **Suggest**: Improvements (1-3 items)
   - **Fix**: Must-fix issues (if any)
4. Keep feedback actionable and concise.