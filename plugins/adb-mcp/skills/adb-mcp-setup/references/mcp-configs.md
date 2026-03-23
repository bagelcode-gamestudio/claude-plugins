# App-Specific MCP Configurations

## Claude Code .mcp.json Configs

`{UV_PATH}` = `which uv` 결과 (예: `/Users/username/.local/bin/uv`)
`{REPO_PATH}` = adb-mcp clone 절대 경로 (예: `/Users/username/src/adb-mcp`)

### Photoshop

```json
{
  "mcpServers": {
    "adobe-photoshop": {
      "command": "{UV_PATH}",
      "args": [
        "run",
        "--with", "fonttools",
        "--with", "python-socketio",
        "--with", "mcp",
        "--with", "requests",
        "--with", "websocket-client",
        "--with", "numpy",
        "{REPO_PATH}/mcp/ps-mcp.py"
      ]
    }
  }
}
```

### Premiere Pro

```json
{
  "mcpServers": {
    "adobe-premiere": {
      "command": "{UV_PATH}",
      "args": [
        "run",
        "--with", "fonttools",
        "--with", "python-socketio",
        "--with", "mcp",
        "--with", "requests",
        "--with", "websocket-client",
        "--with", "pillow",
        "{REPO_PATH}/mcp/pr-mcp.py"
      ]
    }
  }
}
```

### After Effects

```json
{
  "mcpServers": {
    "adobe-after-effects": {
      "command": "{UV_PATH}",
      "args": [
        "run",
        "--with", "fonttools",
        "--with", "python-socketio",
        "--with", "mcp",
        "--with", "requests",
        "--with", "websocket-client",
        "--with", "pillow",
        "{REPO_PATH}/mcp/ae-mcp.py"
      ]
    }
  }
}
```

### Illustrator

```json
{
  "mcpServers": {
    "adobe-illustrator": {
      "command": "{UV_PATH}",
      "args": [
        "run",
        "--with", "fonttools",
        "--with", "python-socketio",
        "--with", "mcp",
        "--with", "requests",
        "--with", "websocket-client",
        "--with", "pillow",
        "{REPO_PATH}/mcp/ai-mcp.py"
      ]
    }
  }
}
```

### InDesign

```json
{
  "mcpServers": {
    "adobe-indesign": {
      "command": "{UV_PATH}",
      "args": [
        "run",
        "--with", "fonttools",
        "--with", "python-socketio",
        "--with", "mcp",
        "--with", "requests",
        "--with", "websocket-client",
        "--with", "pillow",
        "{REPO_PATH}/mcp/id-mcp.py"
      ]
    }
  }
}
```

---

## CEP Plugin Symlink Commands

`{REPO_PATH}` = adb-mcp clone 절대 경로

### macOS

```bash
# CEP 디렉토리 생성
mkdir -p "$HOME/Library/Application Support/Adobe/CEP/extensions"

# After Effects
ln -s {REPO_PATH}/cep/com.mikechambers.ae \
  "$HOME/Library/Application Support/Adobe/CEP/extensions/com.mikechambers.ae"

# Illustrator
ln -s {REPO_PATH}/cep/com.mikechambers.ai \
  "$HOME/Library/Application Support/Adobe/CEP/extensions/com.mikechambers.ai"
```

### Windows (관리자 권한)

```cmd
:: After Effects
mklink /D "%APPDATA%\Adobe\CEP\extensions\com.mikechambers.ae" "{REPO_PATH}\cep\com.mikechambers.ae"

:: Illustrator
mklink /D "%APPDATA%\Adobe\CEP\extensions\com.mikechambers.ai" "{REPO_PATH}\cep\com.mikechambers.ai"
```

**검증**: 심볼릭 링크 생성 후 Adobe 앱 재시작 → 플러그인 패널에서 확인.
