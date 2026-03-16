# Setup: Cursor IDE

Cursor connects to Design Bridge MCP through its built-in MCP support.

## Steps

1. **Open the Design Bridge MCP plugin** in your Framer project. Wait for the green dot (connected).

2. **Copy the MCP URL** from the plugin.

3. **Open Cursor's MCP settings:**
   - Open the command palette: `Cmd+Shift+P` (macOS) or `Ctrl+Shift+P` (Windows/Linux)
   - Search for **"Cursor Settings"** and select the **MCP** option

4. **Add the server configuration.** Open or create `~/.cursor/mcp.json` and add:

```json
{
  "mcpServers": {
    "design-bridge": {
      "command": "npx",
      "args": ["mcp-remote", "YOUR_MCP_URL_HERE"]
    }
  }
}
```

Replace `YOUR_MCP_URL_HERE` with the URL you copied from the plugin.

5. **Refresh MCP servers** in Cursor's MCP settings panel.

6. **Switch Cursor to Agent mode** to access MCP tools.

7. **Verify** by asking Cursor: "List all pages in my Framer project"

## Notes

- The MCP URL is unique to your Framer account and works across all your projects.
- The Design Bridge plugin must be open in Framer for Cursor to communicate with it.
- Cursor must be in **Agent mode** (not Ask or Edit mode) to use MCP tools.
- You only need to configure this once.

## Config File Location

- macOS/Linux: `~/.cursor/mcp.json`
- Windows: `%USERPROFILE%\.cursor\mcp.json`

## Troubleshooting

- **Tools not appearing** — Make sure Cursor is in Agent mode and MCP servers are refreshed.
- **"Plugin is not connected"** — Open the Design Bridge MCP plugin in Framer and wait for the green dot.
- **"npx: command not found"** — Install Node.js from https://nodejs.org/
- **Connection drops** — Make sure the Framer tab with the plugin is visible (not minimized or backgrounded).
