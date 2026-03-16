# Design Bridge MCP - Setup Guides

Choose your AI client to get started:

| AI Client | Difficulty | Guide |
|-----------|-----------|-------|
| **Claude Code** | Easiest (one command) | [Setup Guide](claude-code.md) |
| **Claude Desktop** | Moderate (config file edit) | [Setup Guide](claude-desktop.md) |
| **Cursor IDE** | Moderate (config file edit) | [Setup Guide](cursor.md) |

## Quick Start

1. Install the **Design Bridge MCP** plugin from the [Framer Marketplace](https://framer.com/marketplace)
2. Open the plugin in your Framer project
3. Copy the MCP URL shown in the plugin
4. Follow the setup guide for your AI client above

## How It Works

The plugin creates a secure WebSocket connection between your Framer project and MCP-compatible AI assistants. Your Framer user ID serves as a unique identifier, so the MCP URL remains the same across all your projects — you only need to configure it once.

When you make a request through your AI client, it travels through a secure Cloudflare Worker relay to the open Framer plugin, which executes the action using Framer's Plugin API. The response returns through the same channel.

## Requirements

- The Design Bridge MCP plugin must be **open and connected** (green dot) in Framer
- For Claude Desktop and Cursor: [Node.js](https://nodejs.org/) v18+ is required (for the `mcp-remote` bridge)
- For Claude Code: No additional dependencies

## Need Help?

Open an issue on our [GitHub repository](https://github.com/hellojaywilcox/framer-mcp) or reach out to [@hellojaywilcox](https://x.com/hellojaywilcox) on X.
