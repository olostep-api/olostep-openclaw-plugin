---
name: search
description: Search the web using Olostep for live, up-to-date information. Three tools available — answers (AI-synthesised answers with citations), search_web (live web results with URL, title and description, optionally with page content), and create_map (find pages within a specific site). Use when the user needs current data, comparisons, or facts the AI's training data may not have.
---

# Olostep Search

Search the live web and get answers, search results, or find specific pages within a site. Three search tools for different needs.

## Why this matters

AI training data goes stale. When a user asks "which library is best" or "what does X cost" — they need **today's answer**, not last year's. Olostep searches the live web, synthesises answers with source citations, and can return structured JSON you can use directly in code.

## When to use

- User asks "what is the best X for Y" — needs live rankings, not stale training data
- User wants to compare tools, libraries, or services with current pricing
- User needs a fact that may have changed since the AI's cutoff (versions, status, pricing, team)
- User wants to find specific pages within a documentation site or company site
- User asks a question that starts with "is X still...", "what's the latest...", "has Y changed..."

## Which tool to use

| Need | Tool | Why |
|------|------|-----|
| Broad research question, comparison, fact-check | `answers` | AI-synthesised answer from multiple sources, with citations |
| A list of web results to read or compare | `search_web` | Live results with URL, title and description; set `scrape_results` to also get each page's content |
| Find pages within a specific site | `create_map` | URLs from one site, ranked by relevance to a `search_query` |

## Workflow

**For open research / comparisons / fact-checking:**
1. Use `answers` with the user's question as `task`.
2. If the result needs to be used in code, a table, or a comparison, pass a `json` schema.
3. Present the answer with source URLs — offer to `/scrape` any source for deeper detail.

**For web search results:**
1. Use `search_web` with the query. Set `limit` (1-25) for how many results you want.
2. Returns each result's URL, title and description. Use `include_domains` or `exclude_domains` to narrow the sources.
3. Set `scrape_results` to `true` when you need the content of each result page in the same call.

**For finding pages within a specific site:**
1. Use `create_map` with the site URL and a `search_query`.
2. Set `top_n` to keep only the most relevant URLs, or filter with `include_url_patterns` such as `/docs/**`.
3. Offer to `/scrape` the most relevant result for full content.

## Real developer workflows

**"Find the best library for this"**
> "What's the best React drag-and-drop library in 2026?"
→ `answers` with schema `[{"library": "", "stars": "", "bundle_size": "", "last_release": "", "verdict": ""}]` → structured comparison with live data.

**"What's the current price?"**
> "What do Vercel, Netlify, and Render charge for a Pro plan?"
→ `answers` with schema `[{"provider": "", "pro_price": "", "included": ""}]` → live pricing you can paste into a doc or comparison page.

**"Is this still maintained?"**
> "Is moment.js still maintained? What should I use instead?"
→ `answers` returns current status + recommended alternatives with sources.

**"Find the right docs page"**
> "Search the Supabase docs for row-level security examples"
→ `create_map` on `https://supabase.com/docs` with `search_query` `"row level security"` and `top_n` 5 → ranked URLs → scrape the top one.

**"Get search results for a competitive analysis"**
> "Search for 'best headless CMS 2026' and give me the top 10 results"
→ `search_web` with `limit` 10 → list of titles, URLs and descriptions.

**"Check current version / status"**
> "What's the latest version of Next.js and when was it released?"
→ `answers` → returns current version, release date, key changes, with source link.

## Chain with other skills

- **search → scrape**: Use `answers` or `search_web` to find URLs, then `/scrape` the best ones for full content.
- **search → docs-to-code**: Search for the docs URL, then use `/docs-to-code` workflow to write the integration.
- **search → batch**: Get a list of URLs from search results, then `/batch` scrape all of them.

## Parameters

### `answers` — AI-synthesised answers with citations
- **task**: Question or research task (required)
- **json**: Output schema for structured results (optional but powerful)
  - String: `"return an array with name, price, and pros for each option"`
  - Object: `{"name": "", "price": "", "pros": [], "cons": []}`

### `search_web` — Live web results
- **query**: Search query (required)
- **limit**: Number of results, 1-25
- **include_domains** / **exclude_domains**: Only return, or never return, results from these domains
- **scrape_results**: Set to `true` to also get each result page's content as markdown (uses extra credits per page)

### `create_map` — Find pages within a specific site
- **url**: Site to search within (required)
- **search_query**: What to find, used to rank the URLs
- **top_n**: Keep only the N most relevant URLs
- **include_url_patterns** / **exclude_url_patterns**: Glob patterns such as `/docs/**`

## Tips
- `answers` is the most powerful — it searches the web, reads sources, and returns a synthesised answer with citations
- The `json` param on `answers` lets you get data in a shape you can paste straight into code or a markdown table
- Use `search_web` when you need the actual results (URLs, titles, descriptions) rather than a synthesised answer
- Use `create_map` when the user already knows which site to search — it's faster and more precise than broad web search
- Always offer to `/scrape` cited URLs if the user wants deeper detail on any result
