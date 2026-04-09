---
name: google-search-console
description: "Query Google Search Console data via MCP. Use when the user asks about search queries, impressions, clicks, CTR, average position, SEO performance, keyword rankings, page performance in search, search appearance, quick wins, content gaps, or any Search Console reporting. Trigger on: SEO analysis, search performance, keyword tracking, ranking analysis, CTR optimization, SERP performance."
requires:
  mcp: [gsc]
license: MIT
---

# Google Search Console — SEO Performance Analysis

## Prerequisites & Setup

**Before any query, verify the MCP server is available.** Check if the `search_analytics` tool exists.

**If the tool is NOT available, walk the user through setup:**

### Step 1: Install the MCP Server

Ask the user to create or edit `.mcp.json` in their project root:

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

### Step 2: Get Credentials

Guide the user through these steps:

1. **Google Cloud Console** — Go to https://console.cloud.google.com/
2. **Create or select a project**
3. **Enable the Search Console API** — APIs & Services > Library > search "Google Search Console API" > Enable
4. **Create a service account** — IAM & Admin > Service Accounts > Create Service Account
5. **Generate a JSON key** — click the service account > Keys > Add Key > JSON > Download
6. **Save the JSON file** to a secure location and note the path
7. **Set the path** in `GOOGLE_APPLICATION_CREDENTIALS` in your `.mcp.json`
8. **Grant access in Search Console:**
   - Go to https://search.google.com/search-console/
   - Select your property > Settings > Users and permissions > Add user
   - Add the service account email (from the JSON file's `client_email` field) as **Owner**

### Step 3: Verify

Ask the user to restart Claude Code, then test with: "Show me my top search queries for the last 7 days"

If the MCP server fails to connect, check:
- Node.js 18+ is installed
- The JSON credentials file path is correct and the file exists
- The service account email has Owner access in Search Console
- The property is verified in Search Console

### Property URL

You also need the user's Search Console property URL (ask if not provided):
- URL-prefix property: `https://example.com/`
- Domain property: `sc-domain:example.com`

## Parse the Request

Before calling the tool, extract these from the user's question:

- **siteUrl** — the property URL. Ask the user if unknown.
- **Date range** — map natural language to `startDate` / `endDate` in `YYYY-MM-DD`:
  - "this month" = 1st of current month to today
  - "last month" = 1st to last day of previous month
  - "last 7 days" = 7 days ago to today
  - "last 28 days" = 28 days ago to today (default if unspecified)
  - "last 3 months" = 90 days ago to today
- **Dimensions** — pick from: `query`, `page`, `country`, `device`, `searchAppearance`, `date`
- **Filters** — extract any page, query, country, or device filters mentioned
- **Row limit** — default to 25 for overview, 50 for deep dives, or whatever the user specifies

## Workflows

### 1. Query Performance — Top Search Queries

Use when the user asks about top queries, best keywords, or search terms.

Call `search_analytics` with:
- `dimensions`: `["query"]`
- Date range from user request (default: last 28 days)
- `rowLimit`: 25 (or user-specified)

Present results as a ranked table sorted by clicks descending:

| # | Query | Clicks | Impressions | CTR | Avg Position |
|---|-------|--------|-------------|-----|--------------|

Highlight queries with CTR above 5% as strong performers. Flag queries with high impressions but low clicks as optimization candidates.

### 2. Page Analysis — Top Pages by Search Performance

Use when the user asks about page performance, top landing pages, or URL-level data.

Call `search_analytics` with:
- `dimensions`: `["page"]`
- Date range from user request
- `rowLimit`: 25

Identify:
- **Top performers**: pages with high clicks AND high CTR
- **Underperformers**: pages with high impressions but low CTR (title/description may need work)
- **Rising pages**: if comparing two periods, pages with the biggest click gains

### 3. Quick Wins Detection — Keywords Close to Page 1

Use when the user asks about quick wins, easy gains, low-hanging fruit, or keywords close to page 1.

Option A — Use the built-in detection:
- `detectQuickWins`: `true`

Option B — Manual filter:
- `dimensions`: `["query"]`
- Filter for positions between 4 and 20
- Sort by impressions descending
- `rowLimit`: 50

These are the easiest SEO gains. Keywords ranking 4-20 with high impressions just need a content refresh, better internal linking, or on-page optimization to reach page 1.

Present with clear priority ranking: highest impressions + closest to position 4 = highest priority.

### 4. CTR Optimization — High Impressions, Low CTR

Use when the user asks about CTR issues, underperforming snippets, or title/description optimization.

Call `search_analytics` with:
- `dimensions`: `["query"]` or `["page"]` (depending on context)
- Date range from user request
- `rowLimit`: 50

Filter results for:
- Impressions > median impressions of the result set
- CTR < 2% (or below the dataset average)

For each underperformer, recommend:
- Title tag improvements (include the query, add power words, match intent)
- Meta description updates (add CTA, highlight unique value)
- Schema markup additions (FAQ, How-to, Review) to enhance SERP appearance

### 5. Device Comparison — Mobile vs Desktop Performance

Use when the user asks about mobile performance, device breakdown, or responsive SEO.

Call `search_analytics` with:
- `dimensions`: `["device"]`
- Date range from user request

Present a side-by-side comparison:

| Metric | Desktop | Mobile | Tablet |
|--------|---------|--------|--------|

Flag any mobile-specific issues:
- Mobile CTR significantly lower than desktop = possible UX or page speed issue
- Mobile position worse than desktop = mobile-first indexing may be penalizing the site
- Large impression gap = different query intent by device

### 6. Geographic Analysis — Performance by Country

Use when the user asks about international SEO, country performance, or geographic targeting.

Call `search_analytics` with:
- `dimensions`: `["country"]`
- Date range from user request
- `rowLimit`: 25
- Optionally filter by `countryFilter` if the user specifies a country

Identify:
- Top countries by clicks and impressions
- Countries with high impressions but low CTR (localization opportunity)
- Unexpected geographic traffic (may indicate content resonating in new markets)

## Output Format

Structure every response like this:

```
## Search Console: [site] — [date range]

### Key Findings
- [2-3 most important takeaways]

### [Primary Data Table]
| Column | Column | Column | Column | Column |
|--------|--------|--------|--------|--------|
| data   | data   | data   | data   | data   |

### Quick Wins
[If applicable: keywords in positions 4-20 with optimization potential]

### Recommendations
1. [Actionable SEO item with specific next step]
2. [Actionable SEO item with specific next step]
3. [Actionable SEO item with specific next step]
```

## Caveats

- **Data delay**: Search Console data has a 2-3 day delay. The most recent complete data is typically from 3 days ago.
- **Google Search only**: Metrics cover Google Search results only (not Bing, Yahoo, etc.).
- **CTR varies by SERP features**: Rich results, featured snippets, and ads affect CTR. A "low" CTR may be normal for competitive queries.
- **Position is an average**: A position of 5.3 does not mean you always rank #5 — it is an average across all impressions during the period.

## Limitations

- Cannot access URL inspection data (indexing status, crawl info) — only search analytics
- Cannot modify Search Console settings or submit sitemaps
- Historical data is limited to 16 months
- Anonymous queries (privacy-filtered) are excluded from query-dimension results
- Row limit cap is 25,000 per request — for very large sites, use filters to segment data
