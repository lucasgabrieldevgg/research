# paid-apis — the six worth paying for

> Pricing figures are indicative as of January 2026 and verified against vendor pricing pages where reachable; every vendor changes pricing faster than any skill file can track — treat the numbers as ratios and the decision rules as durable. See ATTRIBUTION.md.

The free toolbox covers encyclopedic, academic, government, community, and code categories completely. What it cannot give you is **general web search** — ranked results across the open web. That gap is the only reason to pay. Everything below is a different flavor of closing it.

## The shortlist

| API | One-liner | Free tier | Paid from (approx.) | Best at |
| --- | --- | --- | --- | --- |
| **Brave Search API** | Independent web index, transparent pricing | ~2,000 queries/mo (1 qps) | ~$3–5 per 1k (Base/Pro/AI tiers) | Cost-efficient grounding with an index that isn't Google's |
| **Tavily** | Web access layer built for agents | ~1,000 credits/mo | ~$8 per 1k credits (plans from ~$30/mo) | Search+extract+crawl in one API, LLM-ready output |
| **Exa** | Neural/semantic search | ~$10 of credits/mo | ~$10 per 1k+ (usage-based) | "Find pages *like* this", high-precision discovery |
| **SerpAPI** | Structured SERP scraping, 15+ engines | 100 searches/mo | ~$50–75/mo for 5k | Vertical engines (Images, News, Scholar) as clean JSON |
| **Serper.dev** | Cheap, fast Google SERP | 2,500 one-time credits | ~$1 per 1k (credit packs) | Highest volume per dollar when quality tier is "good enough" |
| **Perplexity Sonar** | Search + synthesis + citations in one call | Trial credits | Search from ~$5–6 per 1k requests (+ source fees) | When you want the *answer* synthesized, not the results |

## When to pay at all

A decision, not a habit:

```
Does the question need ranked results from the open web
(not encyclopedic/academic/gov/community — those are free)?
├── No → free toolbox. Done.
└── Yes →
    ├── How many queries? (< ~1–2k/mo)
    │     → any free tier above covers it. Done.
    └── More, or production-grade reliability needed?
          ├── Latency-sensitive, cited synthesis wanted → Perplexity Sonar
          ├── Cost-sensitive bulk grounding → Brave (or Serper at higher volume)
          ├── Agent pipelines (search→read→extract loops) → Tavily
          ├── Discovery by meaning, not keywords → Exa
          └── Vertical engines as structured data → SerpAPI
```

**Free-first routing** (the default architecture): a `search()` function that tries the free tier of a paid provider, falls back to the next provider on quota errors, and logs which tier served each query. Costs nothing extra, survives vendor throttling, and produces the exact ledger entry (which provider, which cost) that Stage 3 evaluation wants.

## The six, individually

### Brave Search API — the index-independent workhorse

The only major search index that is neither Google's nor Bing's — independent results, no giant-CoE lock-in, and the pricing is public and linear. Endpoints: `/web/v1/search`, plus AI Snippets (answer synthesis) in the Data for AI plan.

```ts
const r = await fetch("https://api.search.brave.com/res/v1/web/search?q=solid+state+battery+market", {
  headers: {
    "X-Subscription-Token": process.env.BRAVE_API_KEY!,   // env var, never hardcoded
    "Accept": "application/json", "Accept-Encoding": "gzip"
  }
});
const { web: { results } } = await r.json(); // title, url, description, age
```

```python
import os, httpx
r = httpx.get("https://api.search.brave.com/res/v1/web/search",
              params={"q": "solid state battery market", "count": 10},
              headers={"X-Subscription-Token": os.environ["BRAVE_API_KEY"]},
              timeout=15)
for hit in r.json()["web"]["results"]:
    print(hit["title"], hit["url"])
```

Free tier: ~2,000 queries/month, 1 query/second — enough for a month of standard-mode research runs. There is also an official MCP server (see `beyond-apis.md`) if you'd rather not write the client at all.

**Choose it when:** you want a stable, honest-priced general search grounding, or you're already in Brave's ecosystem. **Watch for:** result quality on hyper-local and non-English queries trails Google-based competitors.

### Tavily — the agent-native stack

Purpose-built for LLM pipelines: one API family for `search`, `extract`, `crawl`, and `map`, with relevance-scored, content-first results (the body text, not ten blue links). The research loop's Stage 2→3 handoff — "find pages, then actually read them" — collapses into one call.

```ts
const r = await fetch("https://api.tavily.com/search", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    api_key: process.env.TAVILY_API_KEY,
    query: "EU AI Act enforcement timeline",
    search_depth: "advanced",      // two-pass retrieval; worth it for decision-grade runs
    include_answer: true,           // pre-synthesized answer + sources
    max_results: 10
  })
});
```

```python
import os, httpx
r = httpx.post("https://api.tavily.com/search", json={
    "api_key": os.environ["TAVILY_API_KEY"],
    "query": "EU AI Act enforcement timeline",
    "search_depth": "advanced", "include_answer": True, "max_results": 10}, timeout=30)
d = r.json(); print(d["answer"]); [print(x["url"]) for x in d["results"]]
```

**Choose it when:** the pipeline is search→read→extract→synthesize repeatedly. `search_depth: "advanced"` for load-bearing axes; basic for gap-fills. **Watch for:** credit burn on crawl mode — a crawl of one mid-size site can eat a day's free credits.

### Exa — search by meaning

Embedding-indexed search: "pages like this one", "blog posts arguing X", "technical discussions of Y" — queries where keyword search fails because you don't know the keywords. Also does category filters and "find similar to this URL".

```ts
const r = await fetch("https://api.exa.ai/search", {
  method: "POST",
  headers: { "x-api-key": process.env.EXA_API_KEY!, "Content-Type": "application/json" },
  body: JSON.stringify({
    query: "practitioner essays on monorepo regrets",
    type: "neural", numResults: 10,
    contents: { text: { maxCharacters: 2000 } }   // fetch content with results
  })
});
```

```python
import os, httpx
r = httpx.post("https://api.exa.ai/search",
    headers={"x-api-key": os.environ["EXA_API_KEY"]},
    json={"query": "essays on monorepo regrets", "type": "neural",
          "numResults": 10, "contents": {"text": {"maxCharacters": 2000}}}, timeout=30)
```

**Choose it when:** discovery is the bottleneck (trend-type and domain-type runs; finding the 20 heterogeneous sources no keyword query surfaces). **Watch for:** it replaces keyword search's precision — literal-string lookups still want a classic engine.

### SerpAPI — the vertical-engine Swiss army knife

Scrapes and structures Google (Web, News, Images, Scholar, Patents...) plus other engines, and returns clean JSON with the entire result anatomy (knowledge graph, related questions, pagination). It's the only practical way to get Google **Scholar** and **Patents** as data — legal-type and academic-type runs quietly depend on this.

```ts
const r = await fetch(`https://serpapi.com/search.json?engine=google_scholar&q=spiking+neural+networks+energy&api_key=${process.env.SERPAPI_KEY}&num=10`);
const { organic_results } = await r.json(); // title, snippet, publication, cited_by
```

```python
import os, httpx
r = httpx.get("https://serpapi.com/search.json", params={
    "engine": "google_scholar", "q": "spiking neural networks energy",
    "api_key": os.environ["SERPAPI_KEY"], "num": 10}, timeout=30)
```

**Choose it when:** a specific vertical engine (Scholar, Patents, Images, Google Trends via `google_trends`) is the only source that has it. **Watch for:** the priciest per-query of the six; the free 100/month belongs in the free-first router as "vertical emergencies only".

### Serper.dev — volume per dollar

Google SERP as JSON, ~$1 per 1,000 at pack rates, sub-second latency. No frills — no answer synthesis, no neural anything — but for bulk grounding ("search 500 long-tail entity queries") nothing else is close.

```ts
const r = await fetch("https://google.serper.dev/search", {
  method: "POST",
  headers: { "X-API-KEY": process.env.SERPER_KEY!, "Content-Type": "application/json" },
  body: JSON.stringify({ q: "acme corp founded year", num: 10 })
});
```

```python
import os, httpx
r = httpx.post("https://google.serper.dev/search",
    headers={"X-API-KEY": os.environ["SERPER_KEY"]},
    json={"q": "acme corp founded year", "num": 10}, timeout=15)
```

**Choose it when:** a batch phase needs hundreds of cheap lookups and the quality bar is "Google page one, structured". **Watch for:** SERP scraping is inherently Google-dependent — a layout change degrades all such tools until patched; don't make it the only search path in the system.

### Perplexity Sonar — search as an answered question

One call returns a synthesized, citation-carrying answer instead of results. For research, it's a **second opinion machine**: run the same question through Sonar and through your own pipeline; agreement between two independent syntheses is cheap triangulation.

```ts
const r = await fetch("https://api.perplexity.ai/chat/completions", {
  method: "POST",
  headers: { "Authorization": `Bearer ${process.env.PERPLEXITY_KEY}`, "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "sonar",   // sonar-pro for deeper search passes
    messages: [{ role: "user", content: "What is the current state of EU AI Act enforcement? Cite sources." }]
  })
});
```

```python
import os, httpx
r = httpx.post("https://api.perplexity.ai/chat/completions",
    headers={"Authorization": f"Bearer {os.environ['PERPLEXITY_KEY']}"},
    json={"model": "sonar", "messages": [
        {"role": "user", "content": "Current state of EU AI Act enforcement? Cite sources."}]}, timeout=60)
```

**Choose it when:** time-boxed gap-fills and cross-checks matter more than raw control. **Watch for:** its citations must still enter *your* ledger and be spot-checked — an answered question is not a verified one; and pricing has both per-request and per-source components, so cost models need both numbers.

## Cost discipline (applies to all six)

1. **Env-keyed clients only** — a key in source code is a future leak and an audit failure (`integration.md`).
2. **Budget declared at Stage 1** ("≤500 credits"), tracked per axis; the loop stops spending when the budget dies, not when the card does.
3. **Cache at the query level** — identical queries within a run (and across runs on stable topics) hit disk, not the vendor. Research queries are remarkably repetitive; this alone typically halves spend.
4. **Tier your depths** — basic search for gap-fills, advanced/deep for load-bearing axes; `search_depth` flags exist precisely for this.
5. **Log provider + cost per call** into the source ledger's `kind` field (`paid:tavily`); the report's methodology note then says what the run cost, which is the number that makes the next run's budget rational.
