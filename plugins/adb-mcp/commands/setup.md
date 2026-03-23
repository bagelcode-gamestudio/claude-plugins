---
description: Set up Adobe MCP (adb-mcp) — install, configure, and connect Adobe apps to Claude
argument-hint: <app-name: photoshop|premiere|aftereffects|illustrator|indesign>
---

# Adobe MCP Setup

Run the adb-mcp-setup skill to guide the user through connecting an Adobe app to Claude via MCP.

## Steps

1. If an app name argument is provided, skip the app selection prompt and use it directly.
2. Follow the `adb-mcp-setup` skill workflow exactly — every Step must pass its GATE before proceeding.
3. Use the Quick Reference table in the skill to resolve app-specific values (script, manifest, extra dep).
4. For Claude Code users, read `references/mcp-configs.md` and generate the correct `.mcp.json` with absolute paths detected from the user's environment.
5. Run all prerequisite checks in parallel (python3, node, uv, git).
6. After Step 6 (End-to-End verification) PASS, confirm completion.

## Self-Review

Before marking setup as complete, verify:
- [ ] All 6 GATE checks passed
- [ ] Proxy server running and plugin connected (confirmed via proxy terminal output)
- [ ] MCP tools visible in Claude (test prompt succeeded)
