---
description: Set up Photoshop MCP — install, configure, and connect Photoshop to Claude
argument-hint:
---

# Photoshop MCP Setup

Run the photoshop-mcp-setup skill to guide the user through connecting Photoshop to Claude via MCP.

## Steps

1. Follow the `photoshop-mcp-setup` skill workflow exactly — every Step must pass its GATE before proceeding.
2. Use the reference config in `references/mcp-configs.md` to generate `.mcp.json` with absolute paths.
3. Run all prerequisite checks in parallel (python3, node, uv, git).
4. After Step 6 (End-to-End verification) PASS, confirm completion.

## Self-Review

Before marking setup as complete, verify:
- [ ] All 6 GATE checks passed
- [ ] Proxy server running and plugin connected (confirmed via proxy terminal output)
- [ ] MCP tools visible in Claude (test prompt succeeded)
