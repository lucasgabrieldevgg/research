# integration — wiring research access into a project

> Patterns for the "AI consegue integrar sem complicação" requirement: every pattern here is deliberately the boring, zero-dependency version first, with the robust version as a drop-in upgrade. See ATTRIBUTION.md.

The skill's research loop uses these APIs ad-hoc. But when the deliverable is an *application* that researches — a dashboard, a RAG pipeline, a monitoring bot — the calls need to live in the project. These are the patterns that keep that cheap, testable, and honest about cost.

## Pattern 0 — the env contract

Every keyed API, no exceptions:

```bash
# .env (gitignored — verify with: git check-ignore .env)
BRAVE_API_KEY=...
TAVILY_API_KEY=...
EXA_API_KEY=...
```

```ts
// TS: one module owns the contract
export const env = {
  braveKey: process.env.BRAVE_API_KEY,      // undefined → feature disabled, not crashed
  tavilyKey: process.env.TAVILY_API_KEY,
};
```

```python
# Python: os.environ, read at call time
import os
def brave_enabled() -> bool:
    return bool(os.environ.get("BRAVE_API_KEY"))
```

Rules: keys live in env vars; code reads them and **degrades gracefully when absent** (the free tier or the next provider takes over); `.env` is gitignored and `.env.example` documents the vars without values. A missing key disables a provider — it never throws at import time.

## Pattern 1 — the zero-dependency client (TypeScript)

Node 18+ has `fetch`. This is the whole client for most free APIs:

```ts
type Source = { url: string; title: string; accessed: string; tier: "Primary" | "Established" | "Low" };

export async function wikiSearch(query: string, limit = 10): Promise<Source[]> {
  const url = `https://en.wikipedia.org/w/rest.php/v1/search/page?q=${encodeURIComponent(query)}&limit=${limit}`;
  const res = await fetch(url, {
    headers: { "Api-User-Agent": "myapp/1.0 (contact@myapp.dev)" },
  });
  if (!res.ok) throw new Error(`Wikipedia ${res.status}: ${await res.text()}`);
  const { pages } = await res.json();
  const accessed = new Date().toISOString().slice(0, 10);
  return pages.map((p: any) => ({
    url: `https://en.wikipedia.org/wiki/${encodeURIComponent(p.key)}`,
    title: p.title, accessed, tier: "Established" as const,
  }));
}
```

## Pattern 2 — the zero-dependency client (Python)

```python
import httpx  # or: from urllib.request import ... if truly zero-dep

def wiki_search(query: str, limit: int = 10) -> list[dict]:
    r = httpx.get(
        "https://en.wikipedia.org/w/api.php",
        params={"action": "query", "list": "search", "srsearch": query,
                "srlimit": limit, "format": "json"},
        headers={"User-Agent": "myapp/1.0 (contact@myapp.dev)"},
        timeout=15,
    )
    r.raise_for_status()
    return r.json()["query"]["search"]
```

httpx over requests: timeouts on by default, HTTP/2, and async support in the same import when the project grows.

## Pattern 3 — retry, backoff, and the Retry-After header

Research APIs rate-limit; polite clients survive it:

```ts
export async function fetchWithRetry(url: string, init?: RequestInit, tries = 3): Promise<Response> {
  for (let i = 0; i < tries; i++) {
    const res = await fetch(url, init);
    if (res.status !== 429 && res.status < 500) return res;
    const retryAfter = Number(res.headers.get("retry-after"));
    const waitMs = retryAfter ? retryAfter * 1000 : 2 ** i * 1000;  // 1s, 2s, 4s
    await new Promise(r => setTimeout(r, waitMs));
  }
  return fetch(url, init);  // last try, let the caller see the real error
}
```

```python
import time
def get_with_retry(url: str, *, params=None, headers=None, tries: int = 3):
    for i in range(tries):
        r = httpx.get(url, params=params, headers=headers, timeout=20)
        if r.status_code != 429 and r.status_code < 500:
            return r
        wait = int(r.headers.get("retry-after", 2 ** i))
        time.sleep(wait)
    return r
```

Rules: 429 and 5xx retry; 4xx (except 429) fail fast — retrying a 403 is denial of service against yourself. Respect `Retry-After` when present. Cap tries at 3.

## Pattern 4 — query-level cache

Research queries repeat *a lot* (the same axis re-checked, stable topics re-run). Cache at the query level, keyed by provider+normalized-params:

```python
import hashlib, json, time, pathlib

CACHE = pathlib.Path(".cache/research"); CACHE.mkdir(parents=True, exist_ok=True)
TTL = {"wikipedia": 86400, "openalex": 86400, "gdelt": 1800, "tavily": 3600}

def cached(provider: str, fn, *args, **kwargs):
    key = hashlib.sha256(json.dumps([provider, args, kwargs], default=str).encode()).hexdigest()[:24]
    path = CACHE / f"{provider}-{key}.json"
    if path.exists() and time.time() - path.stat().st_mtime < TTL.get(provider, 3600):
        return json.loads(path.read_text())
    result = fn(*args, **kwargs)
    path.write_text(json.dumps(result, default=str))
    return result
```

TTLs by volatility: encyclopedic 24h+, academic 24h, news minutes-to-an-hour, paid-search ~1h (long enough to dedupe a run's repeated gap-fills, short enough to stay honest). Free providers cache primarily for politeness; paid providers cache primarily for cost — the ledger notes the provider per call, so the run's cost report stays truthful about what was live versus cached.

## Pattern 5 — the provider router (free-first, paid fallback)

The architecture from `paid-apis.md` as one function:

```ts
type SearchResult = { url: string; title: string; snippet: string; provider: string };

export async function webSearch(query: string): Promise<SearchResult[]> {
  // 1. Free tier of the paid provider (if keyed)
  if (env.braveKey) {
    try {
      const r = await fetchWithRetry("https://api.search.brave.com/res/v1/web/search?q=" + encodeURIComponent(query),
        { headers: { "X-Subscription-Token": env.braveKey, Accept: "application/json" } });
      if (r.ok) return toResults((await r.json()).web?.results ?? [], "brave-free");
    } catch { /* fall through */ }
  }
  // 2. Next paid provider (tavily / serper / ...) — same shape
  // 3. Terminal fallback: DuckDuckGo instant answers, or surface the failure
  throw new Error(`No search provider available for: ${query}`);
}
```

Every result carries its `provider` — that string is what the ledger's `kind` field and the cost report are built from.

## Pattern 6 — the cost meter (paid providers)

```python
import os, json, pathlib, datetime

LEDGER = pathlib.Path("research/cost-ledger.jsonl")
COST_PER_1K = {"brave": 0.005, "tavily": 0.008, "exa": 0.010, "serper": 0.001, "serpapi": 0.015, "sonar": 0.006}

def meter(provider: str, units: int = 1) -> None:
    entry = {"t": datetime.datetime.now().isoformat(timespec="seconds"),
             "provider": provider, "units": units,
             "cost": units / 1000 * COST_PER_1K.get(provider, 0)}
    with LEDGER.open("a") as f:
        f.write(json.dumps(entry) + "\n")
    today = sum(e["cost"] for e in map(json.loads, LEDGER.read_text().splitlines())
                if e["t"].startswith(datetime.date.today().isoformat()))
    if today > float(os.environ.get("RESEARCH_DAILY_CAP", "5.00")):
        raise RuntimeError(f"Research daily cap reached: ${today:.2f}")
```

Indicative per-unit prices, overridable by env (they drift; the meter is the pattern, not the price list). The cap throws *before* the call that would breach it — budget enforcement is architectural, not aspirational.

## Pattern 7 — fixtures for tests (no network, no keys)

```python
# tests/fixtures/wikipedia_search_photosynthesis.json — a real response, saved once
import json, pathlib

def load_fixture(name: str) -> dict:
    return json.loads(pathlib.Path(f"tests/fixtures/{name}.json").read_text())

def test_wiki_search_parses(monkeypatch):
    monkeypatch.setattr(httpx, "get", lambda *a, **k: FakeResponse(load_fixture("wikipedia_search_photosynthesis")))
    results = wiki_search("photosynthesis")
    assert results[0]["title"] == "Photosynthesis"
```

Record fixtures by making one real call and saving the response (scrub personalization if any). CI then runs keyless, rate-limit-proof, and deterministic — and the fixtures double as documentation of each API's response shape.

## What NOT to build

- **A scraping framework** — one-page extraction is a library call (trafilatura); site-wide crawling is almost always the wrong research tool and a compliance liability.
- **A generic "API gateway" abstraction** — seven thin functions with their own retry/cache/meter behavior are easier to audit than one clever middleware tower. Unify only when the third provider arrives and the duplication actually hurts.
- **Client SDKs of SDKs** — most research APIs are one endpoint and one auth header; generated clients add version churn, not value.

Ship the boring version; upgrade only the pattern that actually broke in production.
