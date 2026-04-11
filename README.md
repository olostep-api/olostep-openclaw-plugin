# Olostep for OpenClaw

![OpenClaw Plugin](https://img.shields.io/badge/OpenClaw-Plugin-5A45FF)
![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Skills](https://img.shields.io/badge/skills-13-orange)
![MCP Tools](https://img.shields.io/badge/MCP_tools-9-blueviolet)
![CI](https://img.shields.io/badge/CI-integrity_tests-brightgreen)

Give your OpenClaw agent the live web. Scrape any URL to clean markdown, search Google with structured results, crawl entire sites, batch-process up to 10,000 URLs in parallel, and get AI-synthesized answers with citations — all from inside your agent workflow.

This plugin provides **13 skills** for high-level agent workflows and a **9-tool MCP server** for direct tool access. Handles JS rendering, anti-bot bypass, CAPTCHAs, and residential proxies automatically.

---

## Installation

Install from [ClawHub](https://clawhub.com/plugins/olostep):

```bash
clawhub install olostep
```

Or add the MCP server directly to your OpenClaw configuration:

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

Get your free API key at [olostep.com/auth](https://olostep.com/auth) — 500 requests/month, no credit card.

---

## Skills

### Core Data Skills

| Skill | What it does |
|-------|-------------|
| **scrape** | Extract clean markdown, HTML, JSON, or text from any URL. Full browser rendering with anti-bot bypass. Supports geo-targeting, browser actions (click, scroll, type), and pre-built parsers. |
| **search** | Three search tools in one: `answers` for AI-synthesized results, `google_search` for raw SERP data, and `get_website_urls` for finding pages within a specific domain. |
| **crawl** | Follow links from a starting URL and scrape every page discovered. Set `max_pages`, include/exclude URL patterns, and control crawl depth. |
| **batch** | Scrape up to 10,000 URLs in parallel. All pages rendered concurrently. Use `custom_id` to map results back to sources. |
| **map** | Discover every URL on a website instantly. Filter by glob patterns, rank by search query relevance, limit with `top_n`. |
| **answers** | Ask a question, get an AI-synthesized answer from live web sources with citations. Pass a `json` parameter to get structured output in any shape you define. |

### Workflow Skills

| Skill | What it does |
|-------|-------------|
| **research** | Multi-source competitive intelligence and tool comparisons. Combines search, scrape, and batch to deliver structured, cited analysis with a recommendation. |
| **debug-error** | Paste a stack trace. The agent searches the live web for that exact error, scrapes the GitHub issue or StackOverflow thread, and translates it into a fix for your codebase. |
| **docs-to-code** | Scrape real API documentation and write working integration code from what is actually there — not from stale training data. |
| **migrate-code** | Scrape the migration guide and changelog, extract breaking changes, and refactor your code based on the real documentation. |
| **extract-schema** | Turn any unstructured webpage into typed JSON matching a TypeScript interface, JSON schema, or database model. |
| **integrate** | Auto-detect your stack (language, framework, AI toolkit) and write a complete Olostep SDK integration — install, client setup, tool wiring, and verification. |
| **setup** | Configure the Olostep API key and verify the connection. Troubleshooting for common issues. |

---

## MCP Tools

The bundled MCP server (`olostep-mcp`) exposes 9 tools callable by any MCP-compatible agent:

| Tool | Description |
|------|-------------|
| `scrape_website` | Scrape a single URL — markdown, HTML, JSON, or text output |
| `get_webpage_content` | Retrieve a webpage as clean markdown |
| `search_web` | Search the web, get AI-synthesized answers |
| `google_search` | Structured Google SERP data (organic results, PAA, knowledge graph) |
| `answers` | AI-powered answers with citations and optional structured JSON |
| `batch_scrape_urls` | Batch scrape up to 10,000 URLs in parallel |
| `create_crawl` | Crawl a website by following links from a start URL |
| `create_map` | Discover all URLs on a website with filtering |
| `get_website_urls` | Search and retrieve relevant URLs from a specific domain |

---

## Real Developer Workflows

### Debug an obscure error

Your agent searches for the exact error message across GitHub issues and documentation, scrapes the most relevant threads, and delivers a fix — not a generic suggestion, an actual fix based on what people who hit the same error actually did.

### Write code from live docs

AI models hallucinate API parameters and use deprecated methods. The `docs-to-code` skill scrapes the current documentation and writes code from what is really there. No stale training data, no guessing.

### Research before choosing a tool

Comparing ORMs? Evaluating payment providers? The `research` skill searches multiple sources, scrapes actual pricing and feature pages, and returns a structured comparison with a recommendation and citations.

### Ingest an entire docs site for RAG

```
map → discover all URLs on the docs site
batch → scrape every page in parallel as clean markdown
→ feed into your vector store
```

### Extract structured data at scale

```
map → find all product/listing URLs
batch → scrape with a pre-built parser for typed JSON
→ pipe into your database, API, or seed files
```

### Migrate to a new framework version

The `migrate-code` skill scrapes the official migration guide, extracts every breaking change with before/after patterns, and refactors your code based on the real documentation.

---

## SDK Quick Reference

### Node.js / TypeScript

```bash
npm install olostep
```

```typescript
import Olostep from 'olostep';

const client = new Olostep({ apiKey: process.env.OLOSTEP_API_KEY });

// Scrape a page to markdown
const scrape = await client.scrapes.create('https://example.com');
console.log(scrape.markdown_content);

// Batch scrape with custom IDs
const batch = await client.batches.create([
  { url: 'https://example.com/page-1', customId: 'p1' },
  { url: 'https://example.com/page-2', customId: 'p2' },
]);

// Crawl a site
const crawl = await client.crawls.create({
  url: 'https://docs.example.com',
  maxPages: 50,
  maxDepth: 3,
});

// Discover URLs
const map = await client.maps.create({
  url: 'https://example.com',
  topN: 100,
  searchQuery: 'pricing',
});
```

### Python

```bash
pip install olostep
```

```python
from olostep import Olostep

client = Olostep()  # reads OLOSTEP_API_KEY from env

# Scrape
result = client.scrapes.create(url_to_scrape="https://example.com")
print(result.markdown_content)

# Batch
batch = client.batches.create(urls=["https://example.com", "https://example.org"])

# Crawl
crawl = client.crawls.create(start_url="https://docs.example.com", max_pages=50)

# Map
site_map = client.maps.create(url="https://example.com")

# AI Answers with structured output
answer = client.answers.create(
    task="Compare Stripe vs Square pricing",
    json_format={"providers": [{"name": "", "pricing": "", "best_for": ""}]},
)
```

---

## REST API

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

---

## Pre-built Parsers

Use with `parser` to get structured JSON from specific site types:

| Parser | Use case |
|--------|----------|
| `@olostep/google-search` | Google SERP (organic results, knowledge graph, PAA) |
| `@olostep/amazon-it-product` | Amazon product data (price, rating, features) |
| `@olostep/extract-emails` | Extract email addresses from any page |
| `@olostep/extract-calendars` | Extract calendar events |
| `@olostep/extract-socials` | Extract social media profile links |

---

## Framework Integrations

This plugin works with OpenClaw directly. If you are building outside of OpenClaw, Olostep has dedicated SDKs and integrations:

| Framework | Package | What you get |
|-----------|---------|-------------|
| **LangChain (Python)** | `langchain-olostep` | `scrape_website`, `answer_question`, `scrape_batch`, `crawl_website`, `map_website` |
| **LangChain (TypeScript)** | `langchain-olostep` | `OlostepScrape`, `OlostepAnswer`, `OlostepBatch`, `OlostepCrawl`, `OlostepMap` |
| **CrewAI** | `crewai-olostep` | `olostep_scrape_tool`, `olostep_answer_tool`, `olostep_batch_tool`, `olostep_crawl_tool`, `olostep_map_tool` |
| **Mastra** | `@olostep/mastra` | Full Olostep toolset for Mastra agents |
| **MCP (Cursor, Claude Desktop)** | `olostep-mcp` | 9 MCP tools for any MCP-compatible client |
| **Vercel AI SDK** | `olostep` | Create AI tools that wrap scrape and search |
| **OpenAI function calling** | `olostep` | Define `web_search` and `scrape_url` functions |

---

## Testing

The plugin ships with two test suites:

**Static integrity tests** (run on every push and PR):
```bash
pip install pytest
pytest tests/test_skill_integrity.py -v
```

Validates: frontmatter fields, MCP tool references, Python code block syntax, and absence of deprecated parameters across all 13 skill definitions.

**Live API smoke tests** (run on push to main):
```bash
OLOSTEP_API_KEY=your_key pytest tests/test_live_api.py -v --timeout=60
```

Verifies scrape, map, answers, and batch workflows against the live Olostep API.

---

## Project Structure

```
olostep-clawhub-plugin/
  .clawhub/
    plugin.json          # Plugin metadata, keywords, logo reference
  .github/
    workflows/
      integrity.yml      # CI: static checks on PR, live tests on push to main
  skills/
    answers/SKILL.md     # AI-powered answers with citations
    batch/SKILL.md       # Batch scrape up to 10k URLs
    crawl/SKILL.md       # Autonomous website crawling
    debug-error/SKILL.md # Error debugging from live web
    docs-to-code/SKILL.md # Write code from scraped docs
    extract-schema/SKILL.md # Structured data extraction
    integrate/SKILL.md   # Auto-detect stack, write SDK integration
    map/SKILL.md         # URL discovery and site mapping
    migrate-code/SKILL.md # Framework migration from docs
    research/SKILL.md    # Multi-source competitive research
    scrape/SKILL.md      # Single-page scraping
    search/SKILL.md      # Web search (3 modes)
    setup/SKILL.md       # API key configuration
  tests/
    conftest.py          # Shared fixtures, known MCP tools, stale params
    test_skill_integrity.py  # Static validation of all SKILL.md files
    test_live_api.py     # Live API smoke tests
  mcp.json               # MCP server configuration
  LICENSE                 # MIT
  README.md
```

---

## Links

- [Olostep Website](https://olostep.com)
- [API Documentation](https://docs.olostep.com)
- [Get API Key](https://olostep.com/auth) (free, 500 req/month)
- [Node.js SDK](https://docs.olostep.com/sdks/node-js)
- [Python SDK](https://docs.olostep.com/sdks/python)
- [MCP Server on npm](https://www.npmjs.com/package/olostep-mcp)
- [OpenClaw Skill](https://github.com/olostep/olostep-openclaw-skill) (lightweight, skill-only)
- [ClawHub Plugin Page](https://clawhub.com/plugins/olostep)

## License

MIT — see [LICENSE](LICENSE) for details.
