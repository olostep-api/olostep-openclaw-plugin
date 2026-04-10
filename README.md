# Olostep — web search, scraping and crawling for AI

Turn the entire web into LLM-ready context. Olostep integrates web search, scraping, crawling, and AI answers directly into OpenClaw. Essential for agent workflows: debug obscure errors by scraping GitHub issues, automate framework migrations by scraping breaking changes, turn websites into typed JSON schemas, or crawl entire API docs into context. Handles JS rendering, anti-bot, and proxies automatically.

**MCP Servers:** 1 &nbsp;&nbsp;|&nbsp;&nbsp; **Skills:** 13

---

## Installation

Install from [ClawHub](https://clawhub.com/plugins/olostep) or add the MCP server to your OpenClaw configuration:

```json
{
  "mcpServers": {
    "olostep": {
      "command": "npx",
      "args": ["-y", "olostep-mcp"],
      "env": {
        "OLOSTEP_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

Get your free API key at [olostep.com/auth](https://olostep.com/auth) — 100 credits/month, no credit card.

---

## Skills

### answers

> Get AI-powered answers with citations from live web data using Olostep. Use when the user needs up-to-date facts, competitive intelligence, pricing comparisons, structured data from the web, or wants web-sourced answers in a specific JSON shape they can use directly in code or docs.

**Olostep AI Answers** — Ask a question, get an AI-synthesised answer from live web sources with citations and optional structured JSON output you can paste straight into code.

**Why this matters** — This is the most powerful tool in the Olostep toolkit. Instead of scraping individual pages and parsing them yourself, you describe what you want to know and the shape you want the answer in. Olostep searches the web, reads multiple sources, and returns a synthesised, cited answer — optionally as structured JSON.

**When to use:**
- User needs current information like pricing, versions, status, rankings, team, or funding
- User wants structured data from the web in a JSON shape they define
- User is comparing tools, libraries, or services and needs a fair, cited comparison
- User wants to fact-check a claim with current sources
- User asks things like "what's the current...", "is X still...", "compare...", or "find me..."

**Workflow:**
1. Use `answers` with the user's question as `task`
2. Always use the `json` parameter when the user needs structured, comparable, or code-ready output
3. Return the answer with source URLs
4. Offer to `/scrape` any cited source for deeper detail

**The power of `json`** — The `json` parameter transforms answers from a simple Q&A tool into a structured data extraction engine. You define the output shape, and Olostep fills it with live web data.

```
task: "Top 5 alternatives to Segment for product analytics"
json: [{ "name": "", "pricing": "", "key_feature": "", "best_for": "" }]

task: "Company info for Linear"
json: { "ceo": "", "founded": "", "funding_total": "", "employees": "", "tech_stack": [] }

task: "What changed in Next.js 15?"
json: "Return a bullet list of the top 5 changes with one-line descriptions"
```

**Real developer workflows:**

| Scenario | How |
|----------|-----|
| Populate a comparison table in my app | `answers` with JSON for pricing, providers, and limits |
| Is this library still maintained? | `answers` with structured fields: `maintained`, `latest_version`, `released`, `weekly_downloads` |
| Get company data for my CRM | `answers` with structured schema for CEO, founding year, employee count, funding |
| Compare before choosing a tool | `answers` with JSON schema for ORM pros, cons, and serverless fit |
| Research for a blog post | `answers` to get a cited synthesis across multiple blog posts and discussions |
| Get live stats for a feature decision | `answers` with structured stats and sources |
| Tech stack research | `answers` to aggregate from engineering posts, job listings, and talks |

**Chain with other skills:** answers &rarr; scrape &middot; answers &rarr; batch &middot; answers &rarr; code

**Parameters:**
- `task` — question or research task
- `json` — output shape for structured results

**Tips:**
- Always use `json` when output needs to be structured, compared, or used in code
- Share citation URLs
- Offer `/scrape` for deeper detail
- Great for pricing, versions, status, team info, tech stacks, market data, and tool comparisons

---

### batch

> Scrape up to 10,000 URLs in parallel using Olostep. Use when the user has a list of URLs to extract content from at once — product pages, job listings, pricing pages, changelogs, docs, or any set of known pages.

**Olostep Batch Scrape** — Scrape up to 10,000 URLs in parallel. All pages are scraped concurrently with full browser rendering, anti-bot bypass, and residential proxies.

**Why this matters** — Scraping many pages sequentially takes time and often fails due to rate limits, CAPTCHAs, or IP bans. Olostep batch processes them in parallel and returns fully rendered results quickly.

**When to use:**
- User has 3 or more URLs
- User has a list from `/map`
- User wants to compare multiple pricing or product pages
- User wants to build a dataset from a known list of URLs
- User wants to audit or analyse many pages together

**Workflow:**
1. Collect URLs
2. Build an array like `[{ "url": "...", "custom_id": "..." }]`
3. Call `batch_scrape_urls`
4. Use `custom_id` to map results back
5. Default to markdown
6. Synthesise across all results after scraping

**Parameters:**
- `urls_to_scrape` — array of `{url, custom_id?}` objects, 1-10,000
- `output_format` — `markdown` (default), `html`, `json`, `text`
- `wait_before_scraping` — ms to wait for JS rendering, 0-10000
- `country` — country code for geo-targeted content
- `parser` — specialised parser for structured extraction

**Tips:**
- Always use `custom_id` — it makes matching results to sources trivial
- For 1-2 URLs, use `/scrape`. For 3+, always use `/batch`
- Combine with `/map` for the most powerful pattern: discover &rarr; filter &rarr; batch scrape &rarr; synthesise

---

### crawl

> Autonomously crawl an entire website by following links from a starting URL using Olostep.

**Olostep Crawl** — Autonomously discover and scrape an entire website by following links from a start URL. Every page is rendered in a full browser with anti-bot bypass.

**When to use:**
- User wants complete docs ingested
- User wants all blog posts or articles from a site
- User is preparing context for a RAG pipeline or knowledge base
- User wants to audit all pages across a site
- User wants a full picture of a competitor's docs or marketing

**Workflow:**
1. Use `create_crawl` with the starting URL
2. Set `max_pages: 10` by default
3. Use `follow_links: true`
4. Default to markdown
5. Synthesise after crawling

**Parameters:**
- `start_url` — starting URL for the crawl
- `max_pages` — maximum pages to crawl (default 10)
- `follow_links` — whether to follow links (default true)
- `output_format` — `markdown` (default), `html`, `json`, `text`
- `country` — country code for geo-targeted crawling
- `parser` — specialised parser ID

**Tips:**
- Start small — 10 pages is usually enough for a focused docs section
- Synthesise after crawling — don't dump raw content
- Use `/map` first on large sites to understand structure
- For known lists of URLs, `/batch` is better
- For one page, use `/scrape`

---

### debug-error

> Search the web for a specific error message or bug using Olostep. Use when the user pastes an error stack trace, says their code is failing, or asks how to fix a bug that requires looking up recent GitHub issues, StackOverflow threads, or framework documentation.

**Olostep Debug Error** — Turn obscure error messages into working fixes by searching the live web and scraping GitHub issues, StackOverflow, or official docs.

**When to use:**
- User pastes a stack trace or terminal error
- User is facing a bug with a specific library version
- User asks if there is an open GitHub issue
- User is stuck and standard debugging has not worked

**Workflow:**
1. Extract the core error message, framework, and version
2. Use `answers` with the exact issue
3. If a good issue or thread is cited, use `scrape_website` on that URL
4. Explain root cause
5. Provide exact fix

**Parameters:**
- `answers` &rarr; `task`: the exact error message and framework context
- `scrape_website` &rarr; `url_to_scrape`: URL of the GitHub issue or forum thread found via answers, `output_format`: markdown

**Tips:**
- Prioritise GitHub issues and official docs
- Do not dump scraped text raw — translate it into a fix for the user's codebase
- If an issue is marked "open" with no fix, say so and suggest the most upvoted workaround

---

### docs-to-code

> Scrape API documentation or library docs and use them to write working, up-to-date code. Use when the user wants to integrate a third-party API, learn a new library, generate SDK usage examples, write code based on a docs URL, or update code after a library version change.

**Olostep Docs-to-Code** — Turn any documentation URL into working, tested code. Scrape the real docs, understand the API surface, and write the integration.

**Why this matters** — AI models often hallucinate API parameters, use deprecated methods, and miss breaking changes. This skill scrapes the actual current docs and writes code from what is really there.

**When to use:**
- User wants to integrate a third-party API and shares a docs URL
- User says "write me code using [library]"
- User wants examples for a new version
- User wants a typed client or SDK wrapper
- User says "read the docs and write the code"
- User wants to migrate code after a breaking change

**Workflow:**
1. Scrape docs with `scrape_website`
2. If docs span many pages, use `get_website_urls` or `create_map`
3. For full sites, use `create_crawl`
4. Parse endpoints, parameters, auth, shapes, and errors
5. Write exact code from docs
6. Cite the source

**Tips:**
- Always scrape before writing — never assume training data has the current API surface
- Use `/map` first for large APIs
- Prefer reference pages over overview pages
- Include documented error handling

---

### extract-schema

> Scrape a webpage with Olostep and extract specific structured data matching a TypeScript interface, JSON schema, or database model.

**Olostep Extract Schema** — Turn any unstructured webpage into formatted JSON or TypeScript objects. Scrape the site using Olostep's markdown extraction, then parse the data into the requested schema.

**When to use:**
- User provides a URL and a schema/interface
- User wants to populate a DB or create seed data
- User is building a directory or aggregator
- User wants to convert a blog post or article into structured metadata

**Workflow:**
1. Review the requested schema
2. Scrape with `scrape_website`
3. If needed, use `batch_scrape_urls`
4. Parse markdown into the exact schema
5. Output JSON or write to a file

**Tips:**
- Use markdown output — cleanest for LLM parsing
- Use `null` or omit missing values — never hallucinate data
- For many URLs, use batch

---

### integrate

> Automatically integrate the Olostep SDK into the user's codebase. Analyzes the project, detects the language and framework, chooses the right integration pattern, installs the SDK, writes all the code, and verifies it works.

**Olostep Self-Integrating Setup** — Analyze the codebase, detect the stack, and write a complete Olostep integration: SDK installation, client setup, tool wiring, route wiring, and verification.

**When to use:**
- User says "integrate Olostep"
- User wants Olostep scraping/search added to a project
- User is migrating from another scraping provider

**Phases:**

| Phase | What happens |
|-------|-------------|
| **1. Detect** | Language, framework, AI/agent frameworks, existing Olostep usage, project structure |
| **2. Choose pattern** | LangChain &rarr; `langchain-olostep` / CrewAI &rarr; `crewai-olostep` / Mastra &rarr; `@olostep/mastra` / Agent app &rarr; native / Web app &rarr; native / Pipeline &rarr; native |
| **3. API key** | Use passed key, check env, or ask for it. Store in `.env` |
| **4. Install SDK** | `npm install olostep` or `pip install olostep` (+ framework-specific packages) |
| **5. Write code** | Client init, error handling, TypeScript types, comments linking to docs |
| **6. Verify** | Run install, test a scrape, show summary of changes |

**Migrating from another provider?** The `/integrate` skill auto-detects your existing scraping code and rewrites it to use Olostep's SDK — handles scrape, crawl, map, and batch patterns automatically.

---

### map

> Discover and list all URLs on a website using Olostep. Use when the user wants to see what pages exist on a site, find specific sections, audit a site's structure, or get a URL list to feed into batch or crawl.

**Olostep Map** — Discover every URL on a website instantly.

**When to use:**
- User wants to know what pages exist on a site
- User wants blog, docs, API, or pricing URLs
- User wants to understand site structure
- User needs a URL list for batch or crawl
- User says "find all pages about X on this site"

**Workflow:**
1. Use `create_map`
2. Use `search_query` to rank/filter URLs by relevance
3. Use `include_url_patterns` / `exclude_url_patterns` to narrow by path
4. Use `top_n` to limit results
5. Group by site section
6. Suggest next step: scrape, batch, or crawl

**Parameters:**
- `website_url` — URL to map
- `search_query` — keyword to rank/filter URLs by relevance
- `top_n` — max number of URLs to return
- `include_url_patterns` — glob patterns to include, e.g. `["/blog/**", "/docs/**"]`
- `exclude_url_patterns` — glob patterns to exclude

**Tips:**
- Always suggest a next step — map alone isn't useful; the value is in what you do with the URLs
- `search_query` is powerful — best pages float to the top
- Use `include_url_patterns` / `exclude_url_patterns` to be surgical

---

### migrate-code

> Scrape a migration guide, changelog, or breaking changes document using Olostep and automatically update the user's local code.

**Olostep Migrate Code** — Take the pain out of major version upgrades. Scrape the official migration guide, understand the breaking changes, and refactor the codebase automatically.

**When to use:**
- User provides a URL to a migration guide (e.g. React 18 to 19, Pages to App Router, API v2 to v3)
- User asks to update deprecated code to latest patterns
- User wants to audit a file for deprecated methods based on a new release

**Workflow:**
1. Scrape the migration guide URL with `scrape_website`
2. If multi-page, use `get_website_urls` or `create_map` + `batch_scrape_urls`
3. Extract "Before" and "After" patterns, breaking changes, deprecated methods
4. Analyze the user's code files
5. Apply changes, explaining what was updated based on the guide

**Tips:**
- Always scrape the docs before writing any code
- Break complex migrations down step-by-step
- Add code comments linking back to the scraped documentation URL

---

### research

> Do thorough, cited web research to help the user make a decision. Compare tools, libraries, or services. Evaluate competitors. Research tech stacks, market data, or architecture patterns.

**Olostep Research** — Multi-source web research with structured, actionable intelligence. For when the user needs to make a decision and wants real data — not opinions from stale training data.

**When to use:**
- User is choosing between tools, libraries, frameworks, or services
- User wants to understand competitors' features, pricing, or tech stack
- User is evaluating build vs buy
- User needs market data, adoption stats, or trend information
- User wants to fact-check a technical claim
- User says "help me decide", "what should I use", "compare X vs Y"

**Research methodology:**
1. **Quick answer**: `answers` with a `json` schema for structured, cited results
2. **Deep dive**: `scrape_website` on the most relevant sources
3. **Multi-site comparison**: `batch_scrape_urls` on competitors' pages
4. **Site-specific**: `get_website_urls` to find relevant pages within a site
5. **Synthesise and recommend**: Always end with a recommendation

**Tips:**
- Always use a `json` schema — structured output makes comparisons fair
- Always end with a recommendation — users want a decision, not just facts
- For competitive research, scrape actual pricing/features pages
- Share citation URLs so users can verify

---

### scrape

> Scrape a single webpage and extract its content as clean markdown, HTML, JSON, or text using Olostep. Handles JavaScript-rendered pages, SPAs, and bot-protected sites automatically.

**Olostep Scrape** — Extract clean, LLM-ready content from any URL — including JavaScript-heavy SPAs, bot-protected sites, and geo-restricted pages. No Puppeteer, no proxy config, no CAPTCHA solving.

**When to use:**
- User pastes a URL and says "read this", "summarise this", "use this as context"
- User wants to pull content from a docs page to write code
- User wants to extract structured data from a product page, job listing, or pricing page
- User needs page content from a JS-heavy SPA that raw HTTP can't reach
- User wants geo-specific content (e.g. UK pricing vs US pricing)

**Workflow:**
1. Use `scrape_website` with the URL
2. Default to `output_format: markdown`
3. For JS-heavy pages, set `wait_before_scraping: 3000`
4. For structured product data, use `output_format: json` with a `parser`
5. For geo-targeted content, set `country`
6. After scraping, act on the content — write code, extract fields, summarise, compare

**Parameters:**
- `url_to_scrape` — URL to scrape (required)
- `output_format` — `markdown` (default), `html`, `json`, `text`
- `wait_before_scraping` — ms to wait for JS rendering, 0-10000
- `country` — country code for geo-targeted content (e.g. `US`, `GB`, `DE`, `JP`)
- `parser` — specialised parser (e.g. `@olostep/amazon-product`)

**Tips:**
- Always act on scraped content — don't return raw markdown to the user
- `wait_before_scraping: 3000` is usually enough for SPAs; use 5000 for very slow ones
- If you need 3+ URLs, switch to `/batch`
- If you need an entire site, use `/crawl`

---

### search

> Search the web using Olostep for live, up-to-date information. Three tools available — answers (AI-synthesised), google_search (structured SERP), and get_website_urls (find pages within a site).

**Olostep Search** — Search the live web and get answers, structured results, or find specific pages within a site. Three search tools for different needs.

**Why this matters** — AI training data goes stale. When a user asks "which library is best" or "what does X cost" — they need today's answer, not last year's.

**Which tool to use:**

| Need | Tool | Why |
|------|------|-----|
| Broad research, comparison, fact-check | `answers` | AI-synthesised answer from multiple sources, with citations |
| Raw Google results (organic, PAA, knowledge graph) | `google_search` | Structured SERP data when you need the actual search results |
| Find pages within a specific site | `get_website_urls` | Ranked URLs from one domain, sorted by relevance |

**Parameters:**
- **answers**: `task` (required), `json` (output schema, optional but powerful)
- **google_search**: `query` (required), `country` (optional, default `US`)
- **get_website_urls**: `url` (required), `search_query` (required)

**Tips:**
- `answers` is the most powerful — synthesised answer with citations in one call
- The `json` param lets you get data in a shape you can paste straight into code
- Use `google_search` when you need raw SERP rather than a synthesised answer
- Use `get_website_urls` when the user already knows which site to search
- Always offer to `/scrape` cited URLs for deeper detail

---

### setup

> Configure the Olostep API key so the MCP server and all Olostep skills work correctly.

**Steps:**
1. Get an API key at [olostep.com/auth](https://olostep.com/auth) — free, 30 seconds
2. Set the environment variable or configure the MCP server
3. Verify with a test scrape or search

**Troubleshooting:**
- `401 Unauthorized` &rarr; API key is missing or invalid
- `npx not found` &rarr; Node.js is not installed
- Tools not appearing &rarr; restart after adding the MCP config
- Stale npx cache &rarr; run `npx clear-npx-cache` and restart

**Once configured, all skills are available:** `/scrape`, `/search`, `/batch`, `/crawl`, `/map`, `/answers`, `/docs-to-code`, `/research`, `/debug-error`, `/extract-schema`, `/integrate`, `/migrate-code`

---

## MCP Tools

The Olostep MCP server provides 9 tools directly callable by any MCP-compatible agent:

| Tool | Description |
|------|-------------|
| `scrape_website` | Extract content from a single URL as markdown, HTML, JSON, or text |
| `get_webpage_content` | Retrieve a webpage as clean markdown |
| `search_web` | Search the web and get AI-synthesised answers |
| `google_search` | Get structured Google SERP data (organic, PAA, knowledge graph) |
| `answers` | AI-powered answers with citations and optional structured JSON |
| `batch_scrape_urls` | Batch scrape up to 10,000 URLs in parallel |
| `create_crawl` | Autonomously crawl a website by following links |
| `create_map` | Discover all URLs on a website with filtering |
| `get_website_urls` | Search and retrieve relevant URLs from a specific site |

---

## SDK Reference

### Node.js / TypeScript

```bash
npm install olostep
```

```typescript
import Olostep from 'olostep';

const client = new Olostep({ apiKey: process.env.OLOSTEP_API_KEY });

// Scrape
const scrape = await client.scrapes.create('https://example.com');
console.log(scrape.markdown_content);

// Batch
const batch = await client.batches.create([
  { url: 'https://example.com', customId: 'site-1' },
  { url: 'https://example.org', customId: 'site-2' },
]);

// Crawl
const crawl = await client.crawls.create({
  url: 'https://example.com',
  maxPages: 100,
  maxDepth: 3,
});

// Map
const map = await client.maps.create({
  url: 'https://example.com',
  topN: 100,
  searchQuery: 'blog posts',
});

// Answers (REST — not yet in Node SDK)
const res = await fetch('https://api.olostep.com/v1/answers', {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${process.env.OLOSTEP_API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ task: 'What is the pricing of Stripe?' }),
});
```

### Python

```bash
pip install olostep
```

```python
from olostep import Olostep

client = Olostep(api_key="your-api-key")

# Scrape
result = client.scrapes.create(url_to_scrape="https://example.com")
print(result.markdown_content)

# Batch
batch = client.batches.create(urls=["https://example.com", "https://example.org"])

# Crawl
crawl = client.crawls.create(start_url="https://example.com", max_pages=100)

# Map
maps = client.maps.create(url="https://example.com")

# Answers
answer = client.answers.create(task="What is the capital of France?")
print(answer.answer, answer.sources)
```

### REST API

**Base URL:** `https://api.olostep.com/v1`
**Auth:** `Authorization: Bearer <API_KEY>`

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/v1/scrapes` | Scrape a single URL |
| GET | `/v1/scrapes/:id` | Get scrape result |
| POST | `/v1/batches` | Start a batch (up to 10k URLs) |
| GET | `/v1/batches/:id` | Get batch status |
| GET | `/v1/batches/:id/items` | Get batch results |
| POST | `/v1/crawls` | Start a crawl |
| GET | `/v1/crawls/:id` | Get crawl status |
| GET | `/v1/crawls/:id/pages` | Get crawled pages |
| POST | `/v1/maps` | Map a website's URLs |
| POST | `/v1/answers` | Get AI-powered answers |
| GET | `/v1/answers/:id` | Get answer result |
| GET | `/v1/retrieve` | Retrieve content by ID |

### Pre-built Parsers

| Parser | Use case |
|--------|----------|
| `@olostep/google-search` | Google SERP results |
| `@olostep/amazon-it-product` | Amazon product data |
| `@olostep/extract-emails` | Extract email addresses |
| `@olostep/extract-calendars` | Extract calendar events |
| `@olostep/extract-socials` | Extract social media links |

---

## Framework Integration Patterns

| Framework | SDK | Pattern |
|-----------|-----|---------|
| **Next.js + Vercel AI SDK** | `olostep` | Create `lib/olostep.ts` client, add AI tools for search and scrape |
| **Express.js** | `olostep` | Add `/scrape` and `/search` routes |
| **FastAPI** | `olostep` (Python) | Add `/scrape` and `/search` routes with `AsyncOlostep` |
| **LangChain (Python)** | `langchain-olostep` | `scrape_website`, `answer_question`, `scrape_batch`, `crawl_website`, `map_website` |
| **LangChain (TypeScript)** | `langchain-olostep` | `OlostepScrape`, `OlostepAnswer`, `OlostepBatch`, `OlostepCrawl`, `OlostepMap` |
| **CrewAI** | `crewai-olostep` | `olostep_scrape_tool`, `olostep_answer_tool`, `olostep_batch_tool`, `olostep_crawl_tool`, `olostep_map_tool` |
| **OpenAI function calling** | `olostep` | Define `web_search` and `scrape_url` tools, route to Olostep |
| **Data pipeline / script** | `olostep` | `maps.create` &rarr; `batches.create` &rarr; `answers.create` |

---

## Links

- [Olostep Website](https://olostep.com)
- [API Documentation](https://docs.olostep.com)
- [Get API Key](https://olostep.com/auth)
- [Node.js SDK](https://docs.olostep.com/sdks/node-js)
- [Python SDK](https://docs.olostep.com/sdks/python)
- [MCP Server on npm](https://www.npmjs.com/package/olostep-mcp)
- [OpenClaw Extension](https://github.com/olostep/openclaw-olostep-plugin)
- [Cursor Plugin](https://cursor.com/marketplace/olostep)
- [Claude Code Plugin](https://github.com/olostep/olostep-claude-code-plugin)

## License

MIT License — see [LICENSE](LICENSE) for details.
