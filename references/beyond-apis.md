# beyond-apis — MCP servers, datasets, scraping, and feeds

> Package names and versions below were verified against npm/PyPI in January 2026. MCP config examples follow the standard `mcpServers` schema used by Claude Code, GLM, and other MCP hosts. See ATTRIBUTION.md.

APIs are the paved roads. These four methods cover the rest of the territory: MCP when the agent host should call tools natively, datasets when you need bulk data no API serves page-wise, scraping when the source has no API at all, and feeds when the research is longitudinal — watching a topic over weeks rather than snapshotting it once.

## 1. MCP servers — research tools without writing a client

If the environment is an MCP host (Claude Code, GLM, Cursor, or any MCP client), servers give the model first-class tools — no HTTP client code, no key plumbing in your project, just a config block. This is the lowest-friction integration that exists for research.

### The verified research shelf

| Server | Package (verified) | Tools it provides | Key |
| --- | --- | --- | --- |
| Brave Search | `@modelcontextprotocol/server-brave-search` (npm 0.6.2) | `brave_web_search`, `brave_local_search` | `BRAVE_API_KEY` |
| Fetch | `mcp-server-fetch` (PyPI 2026.10.10) | URL → markdown/text, controlled redirects | none |
| Sequential Thinking | `@modelcontextprotocol/server-sequential-thinking` (npm 1.0.0) | structured multi-step reasoning scratchpad | none |
| Memory | `@modelcontextprotocol/server-memory` (npm 1.0.0) | persistent knowledge-graph notes across sessions | none |
| Wikipedia | `wikipedia-mcp` (PyPI 2.0.1) | search, summary, sections, links | none |
| arXiv | `arxiv-mcp-server` (PyPI 0.8.2 / npm 0.1.5) | search, fetch papers, list categories | none |
| PubMed | `pubmedmcp` (PyPI 0.1.4) | PubMed search and fetch | none |
| Tavily | `tavily-mcp` (npm 0.2.22) | `tavily-search`, `tavily-extract`, `tavily-crawl`, `tavily-map` | `TAVILY_API_KEY` |
| Exa | `exa-mcp-server` (npm 3.4.2) | neural search, contents, similar-pages | `EXA_API_KEY` |
| Firecrawl | `firecrawl-mcp` (npm 3.29.1) | crawl/scrape with LLM-ready extraction | `FIRECRAWL_API_KEY` |
| Puppeteer | `@modelcontextprotocol/server-puppeteer` (npm 2025.5.12) | full browser automation for JS-heavy pages | none |

### Configuration (Claude Code / standard MCP host)

`.mcp.json` in the project root (or the host's equivalent):

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    },
    "wikipedia": {
      "command": "uvx",
      "args": ["wikipedia-mcp"]
    },
    "arxiv": {
      "command": "uvx",
      "args": ["arxiv-mcp-server"]
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": { "BRAVE_API_KEY": "${BRAVE_API_KEY}" }
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

Two conventions that matter:

- **Env expansion** (`${BRAVE_API_KEY}`) keeps keys out of files that get committed; where a host doesn't expand env vars, reference the process environment and document it in the project README instead of pasting the key.
- **Prefer uvx/npx run fronts** — no global installs, no version drift; `uvx` runs PyPI servers, `npx -y` runs npm ones.

### How the research loop uses MCP

Map servers to stages: **Gather** uses fetch + the search-family (brave/tavily/exa) + the domain servers (wikipedia/arxiv/pubmed); **Verify/Synthesize** benefits from sequential-thinking as a scratchpad for reconciling conflicts; **Memory** carries the source ledger's key facts across sessions — useful when a research thread spans days. The stage files' citation discipline is unchanged by transport: an MCP-served fact still lands in the ledger with URL, accessed date, and tier.

## 2. Datasets — bulk data no API serves

When the question is aggregate ("how has X changed over 10 years", "train/evaluate on a corpus"), page-wise APIs are the wrong shape. Fetch the bulk:

| Source | What | Access |
| --- | --- | --- |
| **Hugging Face Hub** | 100k+ datasets (research, NLP, evals) | `https://huggingface.co/api/datasets?search=...` (verified; no key for public data), `datasets` library, or `huggingface-cli` |
| **Common Crawl** | Petabytes of crawled web, monthly snapshots | Index API at `index.commoncrawl.org` (⚠ blocked from some datacenter IPs; columnar index on AWS S3, `cdx` tooling) |
| **Internet Archive** | Texts, audio, video, Wayback snapshots | Advancedsearch + metadata APIs (⚠ datacenter-blocked; fine from user IPs); Scholar archives |
| **Kaggle** | Curated competition & community datasets | CLI `kaggle` + free account key |
| **data.gov / data.europa.eu** | Government dataset catalogs | CKAN API; bulk where published |
| **GDELT raw** | The full GKG/event tables behind the news API | BigQuery public dataset — SQL over global news, free tier |
| **Wikimedia dumps** | All of Wikipedia/Wikidata as files | `dumps.wikimedia.org` — the honest way to do bulk wiki analysis |

Dataset etiquette: record the exact snapshot (date + version) in the ledger — a dataset citation without a version is unauditable; prefer the canonical host (the API's own bulk mirror) over third-party re-uploads, which drift.

## 3. Scraping — when there is no API

Rules first, because this is the method with teeth:

1. **`robots.txt` is a contract, not a suggestion.** Check it (`/robots.txt` at the site root) and honor `Disallow` for your agent class.
2. **Rate-limit like a polite human** — 1 request per few seconds, serialized, with a real `User-Agent` including contact.
3. **Prefer the API that's hiding.** Many "no API" sites have JSON endpoints behind their UI (check the network tab pattern `/api/...`) or official RSS. Scraping HTML is the option *after* those fail.
4. **Terms of service exist** — some prohibit automated access; legal-type scrutiny belongs on anything that will be published or commercialized.
5. **Don't rebuild a crawler** when extraction is the need: one page → clean text is a library call, not a crawler.

The one-page extraction stack (verified on PyPI):

```python
# trafilatura: the research-grade main-text extractor
import trafilatura
downloaded = trafilatura.fetch_url("https://example.com/article")
if downloaded:
    text = trafilatura.extract(downloaded, include_comments=False,
                               include_tables=True, favor_recall=True)
    metadata = trafilatura.extract_metadata(downloaded)  # title, author, date!
```

`extract_metadata` is the quiet killer feature for research: **the date and author of a page** — exactly what the ledger's accessed/assessment fields need — extracted alongside the text. Alternative: `readability-lxml` (Python), `@mozilla/readability` (Node). For page → markdown in JS, `r.jina.ai` (free key required since 2024; it's a hosted reader) or Firecrawl (paid/MCP).

When a whole site is needed, keep it bounded: a domain allowlist, a page cap, a depth cap, and a robots.txt check per domain — then feed pages through the same trafilatura call. If the pages are JS-rendered, that's what the Puppeteer MCP server is for.

## 4. RSS/Atom — longitudinal research

Feeds are the only free method that scales *over time*: subscribe once, monitor indefinitely, catch updates the day they publish. For trend-type and community-type runs, the feed layer replaces a standing search budget.

```python
# feedparser: the standard RSS/Atom library (verified on PyPI)
import feedparser
feed = feedparser.parse("https://hnrss.org/newest?q=vector+database&points=100")
for e in feed.entries[:10]:
    print(e.published, e.title, e.link)   # plus e.summary, e.author
```

Useful feed sources that need no key: `hnrss.org` (HN with query/points filters — a ready-made tech-watch service), every major outlet's feed directory, arXiv per-category feeds (`export.arxiv.org/rss/cs.AI`), journal table-of-contents feeds, agency newsrooms (press releases as Primary tier), blog aggregators. The loop's recipe for a monitoring run:

1. **Choose outlets by tier** (from `evaluation.md`) — 3 Established + 2 Primary newsrooms beats 30 random blogs.
2. **Store the subscription list** in the notes directory (`feeds.opml` — feedparser-compatible, portable to any reader).
3. **Poll on a schedule** (daily for standard monitoring; hourly for a live story) with state: last-seen GUID per feed, so each poll is incremental, not a rescrape.
4. **Dedupe by GUID and by title-similarity** across feeds (the normalize-filter-score-dedupe pipeline from `method.md`), score by recency × tier, and append candidates to a `monitor.md` file — which becomes the Gather input for the next full research run on the topic.

The monitoring layer and the snapshot loop share everything downstream: candidates found by monitoring enter the same ledger, the same tier ladder, the same synthesis templates. Monitoring is just Gather with a clock attached.
