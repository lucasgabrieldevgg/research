---
name: research
description: "Research management: run rigorous multi-source research and wire research data sources into projects. Two disciplines in one skill. TRACK 1 - METHOD: deep research as a managed process — frame the question (12 research types: market, domain, technical, competitive, product, academic, person/org, financial, legal, trend, community, social-pulse), plan research axes, gather with parallel fan-out, verify sources with a tier ladder (Primary/Established/Low) and confidence tracking (High requires 2+ independent sources), synthesize with structured notes and fact-vs-synthesis separation, and deliver cited reports with gaps and conflicts stated honestly. TRACK 2 - TOOLBOX: 50+ free research APIs live-verified (Wikipedia/Wikidata, OpenAlex, arXiv, Crossref, Semantic Scholar, Europe PMC, PubMed, GDELT, HN, Stack Exchange, GitHub, World Bank, Open-Meteo, Nominatim...), 6 paid APIs worth the money (Brave Search, Tavily, Exa, SerpAPI, Serper, Perplexity Sonar), MCP servers (wikipedia-mcp, arxiv-mcp, brave-search, fetch, tavily-mcp...), bulk datasets (Hugging Face, Common Crawl, Internet Archive), polite scraping (trafilatura), and RSS/Atom monitoring (feedparser) — each with TypeScript and Python integration patterns. Use this skill for ANY research task: 'research X', 'deep dive', 'literature review', 'what's the state of the art', 'competitive analysis', 'market landscape', 'who are the players', 'catch me up on X', 'monitor this topic', 'find papers on', 'build me a research dashboard/API integration', 'add Wikipedia/arXiv/news data to my app' — even when the user just says 'look into this'. Before delivering any research artifact, run its quality gates."
version: 1.0.0
license: MIT for the synthesis; upstream content keeps its own licenses - see ATTRIBUTION.md
---

# research — deep research as a managed process

`research` works the way a senior analyst would: frame before fetching, evaluate before believing, synthesize before writing — and it knows exactly which free API, paid API, MCP server, dataset, or feed serves each kind of question.

## The research contract

1. **Every claim carries a citation** — URL, accessed date, confidence, tier.
2. **Critical claims need 2+ independent sources**, or are explicitly flagged `confidence: Low`.
3. **No fabrication.** "No sources found for X" is a valid finding; a blank or a guess is not.
4. **No silent conflict resolution.** Disagreement between sources is reported, never quietly adjudicated.
5. **Write-as-you-go.** Every stage writes to disk immediately; nothing batches until the end.
6. **Abstracts are not reading.** Load-bearing sources get actually read, or the report labels itself snippet-based.

## Routing table

Read only what the task needs:

| When | Read |
| --- | --- |
| Running any research task (the core 70%) | `references/method.md` — the 6-stage research loop |
| Judging sources, triangulating, running the ledger | `references/evaluation.md` |
| Deep-reading sources, synthesizing, writing the report | `references/synthesis.md` |
| Choosing and calling free APIs (catalog + code) | `references/free-apis.md` |
| Deciding whether to pay, and which provider | `references/paid-apis.md` |
| MCP servers, bulk datasets, scraping, RSS monitoring | `references/beyond-apis.md` |
| Integrating research access into an app/codebase | `references/integration.md` |

## The loop in one breath

**Frame** (type + depth + recency window; interview only if vague) → **Plan** (axes + source strategy per axis + budget) → **Gather** (fan out per axis; free toolbox first; write each axis file as it completes) → **Verify** (tier ladder, triangulation, conflict protocol; ledger discipline) → **Synthesize** (deep-read the load-bearing 5–15; taxonomy → comparison → gaps; fact/synthesis separation) → **Deliver** (type-specific template, methodology note, freshness note for volatile topics).

Depth modes: **Quick** (narrow, time-boxed — auto-scope, one pass), **Standard** (default — full loop, 5–10 axes), **Deep** (critical decisions — full loop + outline refinement + critique pass + one allowed Gather re-loop).

## Choosing sources — the one-glance matrix

| Question shape | First call |
| --- | --- |
| Definitions, entities, established facts | Wikipedia REST → Wikidata |
| Academic state of the art | OpenAlex → arXiv → Semantic Scholar → Crossref |
| Biomedical | Europe PMC → PubMed |
| News coverage / what happened | GDELT → RSS of tiered outlets |
| Practitioner sentiment | HN Algolia → Stack Exchange → Reddit (MCP/OAuth) |
| Ecosystem maturity | GitHub API → npm/PyPI |
| Economic/statistical backbone | World Bank → FRED → Eurostat → Census |
| Companies, filings, cases | SEC EDGAR → CourtListener → Congress.gov |
| General ranked web results | Paid tier (Brave/Tavily/Exa/Serper/Sonar) or environment WebSearch |

## Hard rules

- Halt and say so if there is no web access at all — do not synthesize from memory and call it research. Offer a knowledge-based briefing, clearly labeled.
- The deep-read mandate is a gate, not a guideline (contract #6).
- Paid API spend follows the declared budget: metered per call, capped daily, logged per source (see `integration.md` Pattern 6).
- MCP-served facts are ledger facts like any other — URL, accessed date, tier, no exceptions.
- Volatile topics (pricing, regulatory, competitive) ship with a freshness note: "snapshot as of {date}; re-run within 30 days".

## Out of scope

- Generating fiction, marketing copy, or any content that doesn't rest on sources (that's the sibling craftsmanship/creative skills' job).
- Long-running production crawlers — this skill scrapes pages, not the web.
- Statistical analysis of gathered data once it's in hand (pandas-type work) — this skill gathers and verifies the data; it doesn't model it.
