# Google Search Console Skill for Claude Code

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Node.js 18+](https://img.shields.io/badge/Node.js-18%2B-green.svg)](https://nodejs.org/)
[![MCP](https://img.shields.io/badge/MCP-Compatible-purple.svg)](https://modelcontextprotocol.io/)

A Claude Code skill for SEO performance analysis. Ask Claude about your search queries, click-through rates, impressions, average positions, and page performance — powered by the Google Search Console API via MCP.

---

## How It Works

- Connects Claude Code to your Search Console property through the `mcp-server-gsc` MCP server
- Claude can query search analytics, find quick-win keywords, analyze page performance, and track position changes
- One tool (`search_analytics`) handles all queries — dimensions, filters, and date ranges let you slice the data any way you need

---

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- Node.js 18+
- A Google Cloud project with the **Search Console API** enabled
- A service account added as **property administrator** in Search Console

---

## Setup — MCP Server (Free & Open Source)

This skill uses [`mcp-server-gsc`](https://github.com/ahonn/mcp-server-gsc) by [ahonn](https://github.com/ahonn).

### Step 1: Create a Google Cloud Service Account

1. Go to [Google Cloud Console](https://console.cloud.google.com/) and create a project (or use an existing one)
2. Enable the **Google Search Console API** (APIs & Services > Library > search "Google Search Console API" > Enable)
3. Create a service account (IAM & Admin > Service Accounts > Create Service Account)
4. Create a JSON key for the service account and download it
5. In [Google Search Console](https://search.google.com/search-console/), go to **Settings > Users and permissions > Add user**
6. Add the service account email (e.g. `my-sa@my-project.iam.gserviceaccount.com`) as **Owner**

### Step 2: Add MCP Server to Claude Code

Create or edit `.mcp.json` in your project root (or `~/.claude/.mcp.json` for global access):

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

Replace `/path/to/your-credentials.json` with the actual path to your downloaded service account key.

### Step 3: Install the Skill

Copy `SKILL.md` to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/google-search-console
cp SKILL.md ~/.claude/skills/google-search-console/SKILL.md
```

Restart Claude Code. The skill is now active.

---

## Alternative Setup — Composio / Rube MCP (Freemium)

If you prefer a managed OAuth flow instead of a service account:

- [Composio](https://composio.dev) — Free tier: 1,000 requests/day. Pro: $25/month. See [pricing](https://composio.dev/pricing).
- [Rube](https://rube.app/mcp) — Managed MCP hosting with Google Search Console support.

These handle authentication for you but introduce a third-party dependency.

---

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

---

## Usage Examples

- "What are my top 20 search queries this month?"
- "Find quick-win keywords — queries ranking positions 4-20 with high impressions"
- "Show me pages with the highest impressions but lowest CTR"
- "Compare my mobile vs desktop search performance"
- "Which queries am I ranking for in France?"
- "Show me pages that lost the most clicks compared to last month"

---

## Credits

- **MCP Server:** [mcp-server-gsc](https://github.com/ahonn/mcp-server-gsc) by [ahonn](https://github.com/ahonn)
- **Alternative MCP:** [Composio](https://composio.dev) / [Rube](https://rube.app/mcp)

---

## Part of the Marketing Suite

| Skill | Repository |
|-------|------------|
| Google Search Console | **You are here** |
| [Google Analytics](https://github.com/judicael-s/google-analytics-skill) | GA4 performance analysis |
| [Google Trends](https://github.com/judicael-s/google-trends-skill) | Trend research and keyword discovery |

---

## License

[MIT](LICENSE) - Jules Sauvajol
