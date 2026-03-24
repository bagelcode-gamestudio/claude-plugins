# Photoshop MCP Configuration

## Claude Code .mcp.json

`{UV_PATH}` = `which uv` 결과 (예: `/Users/username/.local/bin/uv`)
`{REPO_PATH}` = photoshop-mcp clone 절대 경로 (예: `/Users/username/photoshop-mcp`)

```json
{
  "mcpServers": {
    "photoshop-mcp": {
      "command": "{UV_PATH}",
      "args": [
        "run",
        "--with", "mcp",
        "--with", "python-socketio",
        "--with", "fonttools",
        "--with", "requests",
        "--with", "websocket-client",
        "--with", "numpy",
        "{REPO_PATH}/mcp/server.py"
      ]
    }
  }
}
```

## Claude Desktop (auto-register)

```bash
cd {REPO_PATH}/mcp
uv run mcp install \
  --with fonttools --with python-socketio --with mcp \
  --with requests --with websocket-client --with numpy \
  server.py
```

Restart Claude Desktop after install.

## Migration from adb-mcp

If you were using `adb-mcp` for Photoshop, replace:

**Before (adb-mcp):**
```json
{
  "mcpServers": {
    "adobe-photoshop": {
      "command": "{UV_PATH}",
      "args": ["run", "--with", "...", "{REPO_PATH}/mcp/ps-mcp.py"]
    }
  }
}
```

**After (photoshop-mcp):**
```json
{
  "mcpServers": {
    "photoshop-mcp": {
      "command": "{UV_PATH}",
      "args": ["run", "--with", "...", "{REPO_PATH}/mcp/server.py"]
    }
  }
}
```

Key changes:
- Server name: `adobe-photoshop` → `photoshop-mcp`
- Script: `ps-mcp.py` → `server.py`
- Repo: `adb-mcp` → `photoshop-mcp`
- Dependencies: same
