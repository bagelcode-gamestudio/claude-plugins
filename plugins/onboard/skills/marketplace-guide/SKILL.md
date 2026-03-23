---
description: "Auto-activate when user asks about GS plugins, marketplace setup, or available tools in this workspace."
user-invocable: false
globs:
  - ".claude-plugin/**"
  - "**/marketplace.json"
---

# GS Claude Plugins Marketplace Guide

## Available Plugins

| Plugin | Commands | What it does |
|--------|----------|-------------|
| dev | `/dev:slot-server-impl` `/dev:run-sim` | Slot game server development |
| qa | `/qa:test-case` | Test case generation, bug reports |
| ta | `/ta:asset-check` | Asset validation, performance budgets |
| design | `/design:spec-review` | Spec review, data table rules |
| sound | `/sound:audio-check` | Audio checks, naming conventions |
| pm | `/pm:sprint-summary` | Sprint summary, meeting notes |
| adb-mcp | `/adb-mcp:setup` | Adobe MCP setup (Photoshop/Premiere) |

## Quick Start

```bash
# Connect marketplace
/plugin marketplace add bagelcode-gamestudio/claude-plugins

# Install by role
/plugin install dev@claude-plugins

# Or use onboarding command
/onboard:setup dev
```

## When user asks about plugins

- "어떤 플러그인 있어?" → Show the table above
- "설치 방법?" → `/onboard:setup` command
- "XX 플러그인 뭐야?" → Describe that specific plugin
