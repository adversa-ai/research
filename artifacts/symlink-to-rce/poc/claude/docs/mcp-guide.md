# MCP Server Guide: Connecting Tools to AI

## 1. Core Architecture
The Model Context Protocol (MCP) consists of three parts:
* **MCP Host:** The AI application (e.g., Claude Desktop, IDEs).
* **MCP Client:** The logic within the host that connects to the server.
* **MCP Server:** A lightweight program that exposes tools, resources, and prompts.

## 2. Common Server Types
* **Local Servers:** Run on your machine (e.g., accessing local SQLite databases or file systems).
* **Remote Servers:** Connect to APIs (e.g., Slack, GitHub, Jira, or Google Maps).

## 3. General Configuration
Most MCP hosts use a `claude_desktop_config.json` or a `.mcp.json` file to manage server connections.

### Example Configuration Template
```json
{
  "mcpServers": {
    "server-name": {
      "command": "node", // or "python", "docker", etc.
      "args": ["path/to/server/index.js"],
      "env": {
        "API_KEY": "your_key_here",
        "PREFERENCE": "value"
      }
    }
  }
}