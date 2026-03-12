---
description: Data table structure and validation rules for game design sheets
globs:
  - "**/*.csv"
  - "**/*.tsv"
  - "**/*.json"
  - "**/*.xlsx"
---

# Data Table Rules

When creating or reviewing game data tables, enforce these rules:

## Structure
- First row: column headers in snake_case (e.g., `item_id`, `base_damage`)
- ID column required: unique, no gaps, format `{TYPE}_{NUMBER}` (e.g., `WEAPON_001`)
- No empty cells in required columns — use explicit defaults (`0`, `none`, `false`)

## Naming
- Table file: `{system}_{purpose}.csv` (e.g., `weapon_stats.csv`, `reward_schedule.csv`)
- Enum values: UPPER_SNAKE_CASE (e.g., `ELEMENT_FIRE`, `RARITY_EPIC`)
- Reference columns: suffix with `_id` or `_ref` (e.g., `item_id`, `skill_ref`)

## Validation
- Numeric ranges must be defined in a comment row or separate schema
- Foreign key references must point to valid IDs in other tables
- Percentage values: 0-100 (not 0.0-1.0) unless explicitly noted
- Duration values: always specify unit in column name (e.g., `cooldown_sec`, `duration_ms`)

## Change Management
- Version column (`data_version`) required for live-service tables
- Changes to published tables require a changelog entry