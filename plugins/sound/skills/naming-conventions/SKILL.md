---
description: Audio asset naming conventions and FMOD/Wwise integration patterns
globs:
  - "**/*.wav"
  - "**/*.ogg"
  - "**/*.mp3"
  - "**/*.bank"
---

# Sound Naming Conventions

When creating or reviewing audio assets, follow these conventions:

## File Naming
- Pattern: `{category}_{action}_{variant}_{number}`
- Categories: `sfx`, `mus`, `amb`, `vo`, `ui`
- Examples:
  - `sfx_hit_sword_01.wav`
  - `mus_battle_boss_loop.ogg`
  - `amb_forest_day.wav`
  - `vo_npc_greeting_01.wav`
  - `ui_button_click_01.wav`

## FMOD Event Naming
- Path: `event:/{category}/{system}/{action}`
- Examples:
  - `event:/sfx/combat/hit_sword`
  - `event:/mus/battle/boss_theme`
  - `event:/amb/environment/forest`

## Folder Structure
- `/audio/source/` — Original WAV files (not in build)
- `/audio/sfx/` — Sound effects
- `/audio/music/` — Background music
- `/audio/ambience/` — Ambient sounds
- `/audio/voice/` — Voice-over lines

## Mixing Guidelines
- Music: -14 LUFS, ducked -6dB during VO
- SFX: -18 LUFS, prioritize by gameplay importance
- Ambience: -24 LUFS, always looping
- Voice: -12 LUFS, highest priority