---
description: Check audio assets against naming and format conventions
argument-hint: [audio-path]
---

# Audio Check

Validate audio assets against the team's naming and format conventions.

## Instructions

1. If an audio path is given, check that file. Otherwise, check recently added audio assets.
2. Validate against:
   - Naming convention: `{category}_{action}_{variant}` (e.g., `sfx_hit_sword_01`)
   - Format: WAV for source, OGG for in-game (mobile), WAV for in-game (PC)
   - Sample rate: 44.1kHz for music, 22.05kHz for SFX
   - Channels: stereo for music/ambience, mono for SFX
   - Loudness: normalized to -14 LUFS (music), -18 LUFS (SFX)
3. Output format:
   - **Pass**: Files meeting all rules
   - **Warn**: Files approaching limits or with minor issues
   - **Fail**: Files violating conventions with suggested fixes