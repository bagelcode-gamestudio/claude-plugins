---
description: Bug report template and guidelines for consistent issue tracking
globs:
  - "**/*.md"
  - "**/*.txt"
---

# Bug Report Guidelines

When creating or reviewing bug reports, ensure they follow this template:

## Required Fields
- **Title**: `[Module] Short description` (e.g., `[Auth] Login fails with special characters`)
- **Severity**: Critical / Major / Minor / Cosmetic
- **Environment**: Device, OS, app version, build number
- **Steps to Reproduce**: Numbered, specific steps anyone can follow
- **Expected Result**: What should happen
- **Actual Result**: What actually happens
- **Frequency**: Always / Intermittent / Once

## Optional Fields
- **Screenshots/Video**: Attach visual evidence when possible
- **Logs**: Relevant log output or error messages
- **Related Issues**: Link to related bugs or tickets

## Quality Checks
- Steps must be reproducible by someone unfamiliar with the feature
- Avoid vague language ("sometimes", "seems like")
- One bug per report — split compound issues