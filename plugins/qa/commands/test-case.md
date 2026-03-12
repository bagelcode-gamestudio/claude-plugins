---
description: Generate test cases for a feature or code change
argument-hint: <feature-description>
---

# Test Case Generator

Generate structured test cases for the given feature or code change.

## Instructions

1. Analyze the feature description or code diff.
2. Generate test cases covering:
   - Happy path (normal usage)
   - Edge cases (boundary values, empty inputs)
   - Error cases (invalid inputs, failure scenarios)
3. Output format per test case:
   - **ID**: TC-{number}
   - **Category**: Happy / Edge / Error
   - **Precondition**: Setup required
   - **Steps**: Numbered action steps
   - **Expected**: Expected result
   - **Priority**: P0 (critical) / P1 (high) / P2 (medium)
4. Aim for 5-10 test cases per feature.