# Fork Patch Notes

## Why This Fork Exists

This fork fixes a critical bug in `docker mcp` that prevents Claude Code from using dynamically added MCP tools.

## The Bug

When using `mcp-add` to dynamically add an MCP server, tools were registered in `toolRegistrations` (for `mcp-exec`) but NOT in `mcpServer` (for native MCP calls).

**Result:**
```bash
mcp-add chromedev              # ✅ Success, 26 tools registered
mcp-exec chromedev:navigate    # ✅ Works (uses toolRegistrations)
# Native call to chromedev:navigate → ❌ "unknown tool" error
```

## The Fix

**Commit:** `47e9ca32`
**File:** `pkg/gateway/mcpadd.go`

Removed incorrect Claude client exclusion that skipped `updateServerCapabilities()`:

```go
// BEFORE (broken):
if !isClaudeClient(clientInfo) {
    updateServerCapabilities()
}

// AFTER (fixed):
updateServerCapabilities()  // Always update for all clients
```

## Build Instructions

### Prerequisites
- Go 1.24+
- Make

### Build

```bash
# Clone
git clone git@github.com:null-runner/mcp-gateway.git
cd mcp-gateway

# Build
make docker-mcp

# The binary is at:
# - Linux: ./bin/docker-mcp
# - Windows: ./bin/docker-mcp.exe
```

### Install for Claude Code (Windows/WSL)

```powershell
# Copy to user folder
cp bin/docker-mcp.exe $env:USERPROFILE\docker-gateway-mcp-patched.exe
```

Then configure `~/.claude.json`:
```json
{
  "mcpServers": {
    "docker-gateway": {
      "command": "/mnt/c/Users/YOUR_USER/docker-gateway-mcp-patched.exe",
      "args": ["gateway", "run", "--catalog", "docker-mcp.yaml", "--catalog", "custom-servers.yaml"]
    }
  }
}
```

## Upstream Status

This fork tracks `docker/mcp-gateway` upstream. The fix has been submitted but not yet merged.

To sync with upstream:
```bash
git fetch upstream
git merge upstream/main
# Resolve any conflicts in pkg/gateway/mcpadd.go
make docker-mcp
```

## Related Issues

- Original bug discovery: Claude Code tool activation failure
- Fix approach: Remove Claude client special-casing in mcp-add
