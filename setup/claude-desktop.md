# Setup: Claude Desktop

Claude Desktop requires a small bridge tool called `mcp-remote` to connect to Design Bridge MCP. This is a one-time setup.

## Prerequisites

- [Node.js](https://nodejs.org/) (v18 or later) installed on your computer
- Claude Desktop app installed

## Steps

1. **Open the Design Bridge MCP plugin** in your Framer project. Wait for the green dot (connected).

2. **Copy the MCP URL** from the plugin.

3. **Open Claude Desktop settings:**
   - macOS: Claude menu bar > Settings
   - Windows: File > Settings

4. **Navigate to the Developer tab** and click **Edit Config**.

5. **Add the MCP server** to your config file. Replace `YOUR_MCP_URL_HERE` with the URL you copied:

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

If you already have other MCP servers configured, add `"design-bridge"` alongside them inside the `"mcpServers"` object.

6. **Restart Claude Desktop** to activate the connection.

7. **Verify** by asking Claude: "List all pages in my Framer project"

## Notes

- The MCP URL is unique to your Framer account and works across all your projects.
- The Design Bridge plugin must be open in Framer for Claude Desktop to communicate with it.
- `mcp-remote` is automatically downloaded by `npx` on first use — no manual install needed.
- You only need to configure this once. The config persists across restarts.

## Config File Location

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

## Troubleshooting

- **"Plugin is not connected"** — Open the Design Bridge MCP plugin in Framer and wait for the green dot.
- **Tools not appearing in Claude** — Restart Claude Desktop after editing the config.
- **"npx: command not found"** — Install Node.js from https://nodejs.org/
- **Connection drops** — Make sure the Framer tab with the plugin is visible (not minimized or backgrounded).
