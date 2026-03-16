# Setup: Claude Code

Claude Code connects directly to Design Bridge MCP with a single command. No additional dependencies needed.

## Steps

1. **Open the Design Bridge MCP plugin** in your Framer project. Wait for the green dot (connected).

2. **Copy the MCP URL** from the plugin.

3. **Run this command** in your terminal:

```bash
claude mcp add "design-bridge" YOUR_MCP_URL_HERE
```

Replace `YOUR_MCP_URL_HERE` with the URL you copied from the plugin.

4. **Start using it.** The MCP server is now available in all Claude Code sessions. Try:

```
claude "List all pages in my Framer project"
```

## Notes

- The MCP URL is unique to your Framer account and works across all your projects.
- The Design Bridge plugin must be open in Framer for Claude Code to communicate with it.
- You only need to run the `claude mcp add` command once — it persists across sessions.

## Troubleshooting

- **"Plugin is not connected"** — Open the Design Bridge MCP plugin in Framer and wait for the green dot.
- **Tools not responding** — Make sure the plugin tab is visible in Framer (backgrounded tabs may disconnect).
- **Need to update the URL** — Run `claude mcp remove "design-bridge"` then add the new one.
