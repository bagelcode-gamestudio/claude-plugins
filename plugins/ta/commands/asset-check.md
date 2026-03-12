---
description: Check asset files against import rules and performance budgets
argument-hint: [asset-path]
---

# Asset Check

Validate assets against the team's import rules and performance budgets.

## Instructions

1. If an asset path is given, check that specific asset. Otherwise, check recently changed assets.
2. Validate against:
   - Texture size limits (max 2048x2048 for UI, 1024x1024 for in-game)
   - Texture compression format (ASTC for mobile, BC7 for PC)
   - Mesh poly count budgets per LOD level
   - Naming conventions: `{type}_{name}_{variant}` (e.g., `tex_hero_diffuse`)
   - Mipmaps enabled for 3D textures, disabled for UI
3. Output format:
   - **Pass**: Assets meeting all rules
   - **Warn**: Assets approaching limits
   - **Fail**: Assets violating rules with suggested fixes