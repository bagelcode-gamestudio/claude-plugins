---
description: Performance budget guidelines for art assets in Unity
globs:
  - "**/*.fbx"
  - "**/*.png"
  - "**/*.tga"
  - "**/*.psd"
  - "**/*.shader"
  - "**/*.shadergraph"
---

# Performance Budget

When creating or reviewing art assets, enforce these performance budgets:

## Textures
- UI sprites: max 2048x2048, no mipmaps, RGBA 32-bit or compressed
- Character textures: max 1024x1024, mipmaps on, ASTC 6x6 (mobile)
- Environment textures: max 512x512 for props, 1024x1024 for hero assets
- Always use power-of-two dimensions

## Meshes
- Hero characters: max 15K tris
- NPCs/mobs: max 5K tris
- Props: max 1K tris
- LOD0 → LOD1: 50% reduction, LOD1 → LOD2: 50% reduction

## Shaders
- Max 2 texture samples per pass on mobile
- Avoid branching in fragment shaders
- Use shader variants sparingly (max 4 per material)

## Scene Budgets
- Draw calls: max 200 per frame (mobile)
- SetPass calls: max 80
- Total on-screen tris: max 300K