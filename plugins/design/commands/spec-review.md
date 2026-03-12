---
description: Review a game system spec for completeness and consistency
argument-hint: <spec-file-path>
---

# Spec Review

Review a game system specification document for completeness and consistency.

## Instructions

1. Read the provided spec document.
2. Check for:
   - Missing sections (overview, flow, edge cases, dependencies)
   - Undefined terms or references
   - Inconsistent numbers or formulas
   - Missing fail-safe / fallback behavior
   - UX edge cases (what happens on timeout, disconnect, etc.)
3. Output format:
   - **Complete**: Sections that are well-defined
   - **Missing**: Required sections not found
   - **Inconsistent**: Contradictions or ambiguous definitions
   - **Questions**: Clarifying questions for the designer
4. Be specific — reference exact sections and values.