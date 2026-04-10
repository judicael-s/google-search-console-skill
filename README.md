# Google Search Console — MCP Server & AI Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Node.js 18+](https://img.shields.io/badge/Node.js-18%2B-green.svg)](https://nodejs.org/)
[![MCP](https://img.shields.io/badge/MCP-Compatible-purple.svg)](https://modelcontextprotocol.io/)

Query your Google Search Console data from any MCP-compatible client — search queries, click-through rates, impressions, average positions, quick wins, and page performance. Works with Claude Desktop, Claude Code, Cursor, Windsurf, Cline, and any tool supporting the [Model Context Protocol](https://modelcontextprotocol.io/).

## How It Works

This project packages the [`mcp-server-gsc`](https://github.com/ahonn/mcp-server-gsc) MCP server with ready-to-use configuration and an optional Claude Code skill file for guided SEO workflows.

The MCP server connects to the Search Console API using a Google service account and exposes a powerful `search_analytics` tool that any MCP client can call.

## Prerequisites

- **Node.js 18+**
- A Google Cloud project with the **Search Console API** enabled
- A service account added as **property administrator** in Search Console

## Setup

### Step 1: Create a Google Cloud Service Account

1. Go to [Google Cloud Console](https://console.cloud.google.com/) and create a project (or use an existing one)
2. Enable the **Google Search Console API** (APIs & Services > Library > search "Google Search Console API" > Enable)
3. Create a service account (IAM & Admin > Service Accounts > Create Service Account)
4. Create a JSON key for the service account and download it
5. In [Google Search Console](https://search.google.com/search-console/), go to **Settings > Users and permissions > Add user**
6. Add the service account email (e.g. `my-sa@my-project.iam.gserviceaccount.com`) as **Owner**

### Step 2: Add MCP Server to Your Client

<details open>
<summary><strong>Claude Desktop</strong> — <code>claude_desktop_config.json</code></summary>

```json
{
  "mcpServers": {
    "gsc": {
      "command": "npx",
      "args": ["-y", "mcp-server-gsc"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/your-credentials.json"
      }
    }
  }
}
```

Config location: `%APPDATA%\Claude\claude_desktop_config.json` (Windows) or `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS)

</details>

<details>
<summary><strong>Claude Code</strong> — <code>.mcp.json</code> (project root)</summary>

```json
{
  "mcpServers": {
    "gsc": {
      "command": "npx",
      "args": ["-y", "mcp-server-gsc"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/your-credentials.json"
      }
    }
  }
}
```

</details>

<details>
<summary><strong>Cursor / Windsurf / Cline</strong></summary>

Use the same JSON structure above in your editor's MCP configuration:
- **Cursor:** `.cursor/mcp.json`
- **Windsurf:** `~/.codeium/windsurf/mcp_config.json`
- **Cline:** VS Code settings > Cline MCP Servers

</details>

Replace `/path/to/your-credentials.json` with the actual path to your downloaded service account key.

### Step 3: Verify

Restart your MCP client and ask: *"Show me my top search queries for the last 7 days"*

## Available Tool

The MCP server exposes one powerful tool: **`search_analytics`**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `siteUrl` | Yes | Your property URL (`https://example.com/` or `sc-domain:example.com`) |
| `startDate` | Yes | Start date in `YYYY-MM-DD` format |
| `endDate` | Yes | End date in `YYYY-MM-DD` format |
| `dimensions` | No | `query`, `page`, `country`, `device`, `searchAppearance`, `date` |
| `searchType` | No | `web`, `image`, `video`, `news`, `discover`, `googleNews` |
| `rowLimit` | No | 1 - 25,000 (default: 1000) |
| `pageFilter` | No | URL filter (supports `regex:` prefix) |
| `queryFilter` | No | Query filter (supports `regex:` prefix) |
| `countryFilter` | No | ISO 3166-1 alpha-3 country codes (e.g. `FRA`, `USA`) |
| `deviceFilter` | No | `DESKTOP`, `MOBILE`, `TABLET` |
| `filterOperator` | No | `equals`, `contains`, `notEquals`, `notContains`, `includingRegex`, `excludingRegex` |
| `detectQuickWins` | No | Boolean — find keywords close to ranking on page 1 |

### Response Format

```json
{
  "rows": [
    {
      "keys": ["search query"],
      "clicks": 451,
      "impressions": 577,
      "ctr": 0.78,
      "position": 1.2
    }
  ]
}
```

## Usage Examples

Ask your AI assistant in natural language:

- "What are my top 20 search queries this month?"
- "Find quick-win keywords — queries ranking positions 4-20 with high impressions"
- "Show me pages with the highest impressions but lowest CTR"
- "Compare my mobile vs desktop search performance"
- "Which queries am I ranking for in France?"
- "Show me pages that lost the most clicks compared to last month"

## Use as a Claude Code Skill

This repo includes a `SKILL.md` file that turns it into a [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) skill with 6 guided SEO workflows and an interactive setup assistant.

```bash
mkdir -p ~/.claude/skills/google-search-console
cp SKILL.md ~/.claude/skills/google-search-console/SKILL.md
```

The skill will walk users through MCP server setup if it's not already configured.

## Alternative MCP — Composio / Rube (Freemium)

If you prefer a managed OAuth flow instead of a service account:

- [Composio](https://composio.dev) — Free tier: 1,000 requests/day. Pro: $25/month. See [pricing](https://composio.dev/pricing).
- [Rube](https://rube.app/mcp) — Managed MCP hosting with Search Console support.

## Credits

- **MCP Server:** [mcp-server-gsc](https://github.com/ahonn/mcp-server-gsc) by [ahonn](https://github.com/ahonn)
- **Alternative MCP:** [Composio](https://composio.dev) / [Rube](https://rube.app/mcp)

## Part of the Marketing Suite

| Tool | Repository |
|------|------------|
| Google Trends | [judicael-s/google-trends-skill](https://github.com/judicael-s/google-trends-skill) |
| Google Analytics | [judicael-s/google-analytics-skill](https://github.com/judicael-s/google-analytics-skill) |
| **Google Search Console** | **This repo** |
| Copywriting | [judicael-s/copywriting-skill](https://github.com/judicael-s/copywriting-skill) |

## License

[MIT](LICENSE)
