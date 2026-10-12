# free-apis — the zero-cost research toolbox

> Endpoints in this file were live-verified in January 2026 (status, latency, response shape). Entries marked ⚠ were unreachable from the verification sandbox's datacenter IP but are established free services — test from your own environment. See ATTRIBUTION.md.

Selection criteria, in order: **free** (no paid tier required for research-scale use) → **low friction** (no key, or a free key in under two minutes) → **structured output** (JSON preferred) → **tolerant limits** (research-scale volume, not production-scale).

The pattern for every entry: what it answers, the call (TypeScript and Python), and the etiquette that keeps it working. All snippets assume error handling per `integration.md`.

## The 15 that carry most research

### 1. Wikipedia REST + Action API — the encyclopedic backbone

No key, no signup, JSON, generous limits. Three surfaces, one knowledge base:

```ts
// Search (REST, modern)
const r = await fetch("https://en.wikipedia.org/w/rest.php/v1/search/page?q=photosynthesis&limit=5");
const { pages } = await r.json();

// Card-style summary (perfect for entity briefs)
const s = await fetch("https://en.wikipedia.org/api/rest_v1/page/summary/Photosynthesis");
const card = await s.json(); // title, extract, thumbnail, description

// Polyglot: swap the subdomain — pt.wikipedia.org, de.wikipedia.org...
```

```python
import httpx
r = httpx.get("https://en.wikipedia.org/w/api.php", params={
    "action": "query", "list": "search", "srsearch": "photosynthesis",
    "srlimit": 5, "format": "json"}, timeout=15)
for hit in r.json()["query"]["search"]:
    print(hit["title"], hit["snippet"][:80])
```

Etiquette: identify yourself with a descriptive `User-Agent` including contact; the REST API asks for `Api-User-Agent`. Rate: be polite, serialize bursts. For bulk (>100 pages/s) use DB dumps instead.

Research use: entity grounding, definition settling, "related pages" via `morelike:` search, cross-language confirmation. Wikipedia is **Established tier** — a starting point that points at Primary sources, not an endpoint.

### 2. Wikidata SPARQL — structured facts about anything

The machine-readable twin: entities, properties, and relations, queryable.

```
# Countries in South America with their GDP (property P2132... actually nominal GDP = P2132? verify per query)
SELECT ?country ?countryLabel ?gdp WHERE {
  ?country wdt:P31 wd:Q3624078 .         # instance of: sovereign state
  ?country wdt:P36 wd:? .                # filter as needed
  ?country wdt:P2132 ?gdp .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
} LIMIT 20
```

```ts
const q = encodeURIComponent(`SELECT ?item ?itemLabel WHERE {
  ?item wdt:P31 wd:Q5; wdt:P106 wd:Q82594. SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
} LIMIT 10`);
const r = await fetch(`https://query.wikidata.org/sparql?format=json&query=${q}`,
  { headers: { "User-Agent": "research-bot/1.0 (contact@example.com)" } });
const bindings = (await r.json()).results.bindings;
```

```python
import httpx
r = httpx.get("https://query.wikidata.org/sparql", params={"format": "json", "query": QUERY},
              headers={"User-Agent": "research-bot/1.0 (you@example.com)"}, timeout=60)
```

Etiquette: 60-second query timeout; UA with contact **mandatory**; cache results (entity data changes slowly). Property explorer: query.wikidata.org has a built-in helper.

Research use: population lists, filterable inventories ("all companies founded before 1990 in sector X"), fact extraction for ledger Primary entries.

### 3. OpenAlex — the academic mega-index

~250M works, 90M+ authors, citation graphs, concepts, institutions, funding — all free, no key (add `mailto` for the polite pool):

```ts
const mailto = "you@example.com";
const works = await fetch(
  `https://api.openalex.org/works?search=transformer+attention&per-page=10&mailto=${mailto}`);
const { results, meta } = await works.json();
// results[i]: title, publication_year, cited_by_count, doi, primary_location...

// Filtered: highly-cited, 2020+, open access
const f = await fetch(
  `https://api.openalex.org/works?filter=cites:W2963403871,is_oa:true,from_publication_date:2020-01-01&sort=cited_by_count:desc&mailto=${mailto}`);
```

```python
import httpx
r = httpx.get("https://api.openalex.org/works", params={
    "search": "large language models agents",
    "per-page": 25, "mailto": "you@example.com"}, timeout=20)
for w in r.json()["results"]:
    print(w["publication_year"], w["cited_by_count"], w["title"][:70])
```

Entities: `/works`, `/authors`, `/institutions`, `/venues`, `/publishers`, `/concepts` — same API. Etiquette: 10 req/s, 100k/day; datacenter IPs can get throttled (429) — the `mailto` polite pool fixes most of it. ⚠ verified unreachable from the sandbox IP; works from residential/office IPs and CI with mailto.

Research use: the first call of any academic-type run. Citation counts for the academic sub-ladder; `cites:` filter for "who built on X"; `referenced_works` for backward citation walks.

### 4. arXiv — the preprint firehose

No key, Atom XML, 1 request per 3 seconds:

```ts
const q = encodeURIComponent('all:"diffusion models" AND submittedDate:[202501010000 TO 202601010000]');
const r = await fetch(`http://export.arxiv.org/api/query?search_query=${q}&max_results=10&sortBy=submittedDate&sortOrder=descending`);
const xml = await r.text(); // parse entries: title, summary, authors, links, categories
```

```python
import httpx, xml.etree.ElementTree as ET
r = httpx.get("http://export.arxiv.org/api/query", params={
    "search_query": 'all:"graph neural networks"', "max_results": 10,
    "sortBy": "relevance"}, timeout=20)
ns = {"a": "http://www.w3.org/2005/Atom"}
for e in ET.fromstring(r.text).findall("a:entry", ns):
    print(e.find("a:id", ns).text, e.find("a:title", ns).text.strip()[:70])
```

Research use: frontier scans (<3 months old — preprints are the *only* source for the newest layer of academic-type research; every citation carries `(preprint)`). Field queries: `ti:`, `abs:`, `cat:cs.LG`, `au:`.

### 5. Crossref — DOI and publication metadata

130M+ records, free, no key:

```ts
const r = await fetch("https://api.crossref.org/works?query=battery+solid+electrolyte&rows=10&select=title,author,issued,DOI,is-referenced-by-count");
const items = (await r.json()).message.items;
```

```python
import httpx
r = httpx.get("https://api.crossref.org/works", params={
    "query": "perovskite solar efficiency", "rows": 10,
    "select": "title,DOI,issued,is-referenced-by-count"},
    headers={"User-Agent": "research/1.0 (mailto:you@example.com)"}, timeout=20)
```

Research use: turning any citation string into a verified DOI + metadata; reference counting; funder analysis (`filter=has-funder:true`). Polite pool = mailto in the UA.

### 6. Semantic Scholar — citation-aware ML/AI search

No key for the shared pool (100 req/5 min — bursty 429s are the pool, not you; retry with backoff). Free key: 1 req/s sustained:

```ts
const r = await fetch("https://api.semanticscholar.org/graph/v1/paper/search?query=llm+tool+use&limit=20&fields=title,year,citationCount,venue,abstract,authors");
const { data } = await r.json();
```

```python
import httpx, time
def s2_search(query, limit=20):
    for attempt in range(3):
        r = httpx.get("https://api.semanticscholar.org/graph/v1/paper/search",
                      params={"query": query, "limit": limit,
                              "fields": "title,year,citationCount,venue"},
                      timeout=20)
        if r.status_code != 429: return r.json()["data"]
        time.sleep(2 ** attempt)
```

Research use: citation graphs (`/citations`, `/references` per paper), TLDR summaries (`fields=tldr`), author influence. Best ML/AI coverage of the academic set.

### 7. Europe PMC + PubMed — the biomedical pair

Both free, no key at research volume. Europe PMC (JSON-first, full text for open access):

```ts
const r = await fetch("https://www.ebi.ac.uk/europepmc/webservices/rest/search?query=malaria%20vaccine%20efficacy&format=json&pageSize=20&resultType=core");
const { resultList: { results } } = await r.json();
```

```python
import httpx
r = httpx.get("https://www.ebi.ac.uk/europepmc/webservices/rest/search",
              params={"query": "malaria vaccine efficacy", "format": "json",
                      "pageSize": 20, "resultType": "core"}, timeout=20)
```

PubMed E-utilities (the canonical biomedical index):

```python
r = httpx.get("https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
              params={"db": "pubmed", "term": "CRISPR sickle cell", "retmax": 20,
                      "retmode": "json", "sort": "relevance"}, timeout=20)
ids = r.json()["esearchresult"]["idlist"]
# then efetch/esummary for details; 3 req/s without key, more with one
```

Research use: biomedical-type runs; Europe PMC's `resultType=core` returns structured abstracts; check `hasTextMinedTerms` for extracted entities.

### 8. GDELT — the global news index

No key; scans worldwide news coverage in 15-minute batches across 100+ languages:

```ts
const q = encodeURIComponent('"electric vehicle" tariff');
const r = await fetch(`https://api.gdeltproject.org/api/v2/doc/doc?query=${q}&mode=artlist&maxrecords=25&sort=datedesc&format=json`);
const { articles } = await r.json(); // url, title, seendate, domain, language, sourcecountry
```

```python
import httpx
r = httpx.get("https://api.gdeltproject.org/api/v2/doc/doc", params={
    "query": '"renewable energy" brazil', "mode": "artlist",
    "maxrecords": 25, "sort": "datedesc", "format": "json"}, timeout=30)
```

Also `mode=timelinevol` (volume over time — trend lines with zero effort). Etiquette: bursty 429s happen — space calls. Research use: news-coverage maps, media-attention timelines, event discovery inside recency windows; always fetch the article behind each result before citing (GDELT is an index, not a source).

### 9. Hacker News (Algolia) — tech community pulse

No key, instant full-text across posts and comments, generous:

```ts
const r = await fetch("https://hn.algolia.com/api/v1/search?query=rust%20rewrite&tags=story&numericFilters=points>100");
const { hits } = await r.json(); // title, url, points, num_comments, created_at, objectID
```

```python
import httpx
r = httpx.get("https://hn.algolia.com/api/v1/search", params={
    "query": "vector database", "tags": "story",
    "numericFilters": "points>50"}, timeout=15)
```

Research use: practitioner sentiment (Low tier but high-signal for technical-type runs), launch reaction sampling, `search_by_date` for recency windows, `item/{id}` for full comment trees.

### 10. Stack Exchange — the Q&A knowledge base

No key at low volume (300/day anonymous; free key raises it):

```ts
const r = await fetch("https://api.stackexchange.com/2.3/search/advanced?q=docker+networking&site=stackoverflow&pagesize=25&order=desc&sort=relevance");
const { items } = await r.json(); // title, score, answer_count, tags, is_answered
```

```python
import httpx
r = httpx.get("https://api.stackexchange.com/2.3/search/advanced", params={
    "q": "docker networking", "site": "stackoverflow",
    "pagesize": 25, "order": "desc", "sort": "relevance"}, timeout=15)
```

Research use: what practitioners actually struggle with (a demand-side signal no analyst report has), accepted-answer quality as a proxy for topic maturity; 100+ sites (serverfault, stats, etc.) via the `site` param.

### 11. GitHub API — the code ecosystem

No key: 60 req/hr + 10 search/min. Free OAuth token: 5000/hr + 30 search/min:

```ts
const r = await fetch("https://api.github.com/search/repositories?q=topic:llm-agents&sort=stars&order=desc&per_page=10",
  { headers: { "Accept": "application/vnd.github+json", "User-Agent": "research/1.0" } });
const { items } = await r.json(); // full_name, stargazers_count, pushed_at, license, language
```

```python
import httpx
r = httpx.get("https://api.github.com/search/repositories", params={
    "q": "topic:llm-agents", "sort": "stars", "order": "desc", "per_page": 10},
    headers={"Accept": "application/vnd.github+json"}, timeout=15)
```

Research use: ecosystem maturity (stars-over-time via stargazers history), maintenance health (`pushed_at`, issues ratio), license risk (field is right there), code-search for how libraries are actually used. `rest` code search needs auth.

### 12. Open Library + Internet Archive — the book layer ⚠

Both free, no key; both verified unreachable from the sandbox datacenter IP but established services:

```ts
const r = await fetch("https://openlibrary.org/search.json?q=clean+code&limit=10&fields=title,author_name,first_publish_year,subject");
const { docs } = await r.json();

// Internet Archive full-text search
const a = await fetch("https://archive.org/advancedsearch.php?q=collection:opensource+AND+subject:compilers&rows=10&output=json");
```

```python
import httpx
r = httpx.get("https://openlibrary.org/search.json", params={
    "q": "design patterns", "limit": 10,
    "fields": "title,author_name,first_publish_year"}, timeout=20)
```

Research use: bibliographies, publication timelines, borrowing links for verification; Internet Archive's Wayback Machine (separate API) is the citation-rot repair kit — a dead URL gets its archived snapshot cited as "via Wayback {timestamp}".

### 13. World Bank — development indicators

No key, clean JSON:

```ts
const r = await fetch("https://api.worldbank.org/v2/country/BRA;USA;CHN/indicator/NY.GDP.MKTP.CD?format=json&per_page=100&date=2015:2025");
const [meta, data] = await r.json(); // data[i].date, .value
```

```python
import httpx
r = httpx.get("https://api.worldbank.org/v2/country/BRA/indicator/NY.GDP.MKTP.CD",
              params={"format": "json", "per_page": 60, "date": "2015:2025"}, timeout=15)
for d in r.json()[1]:
    print(d["date"], d["value"])
```

Research use: economic backbones for market/domain runs — GDP, population, unemployment, sector shares across 200+ countries and 60 years. Primary tier by definition.

### 14. Open-Meteo — weather and climate data

No key, 10k calls/day non-commercial:

```ts
const r = await fetch("https://api.open-meteo.com/v1/forecast?latitude=-23.55&longitude=-46.63&daily=temperature_2m_max,temperature_2m_min&past_days=90&timezone=America/Sao_Paulo");
const { daily } = await r.json();
```

```python
import httpx
r = httpx.get("https://api.open-meteo.com/v1/forecast", params={
    "latitude": -23.55, "longitude": -46.63, "daily": "temperature_2m_max",
    "past_days": 90, "timezone": "America/Sao_Paulo"}, timeout=15)
```

Research use: environmental context for domain runs; historical weather (`archive` and `historical-weather` endpoints go back decades — agricultural, logistics, and event research).

### 15. Nominatim / OpenStreetMap — the geographic layer ⚠

Free, no key, 1 req/s, contact-info UA mandatory; verified blocked from datacenter IP:

```ts
const r = await fetch("https://nominatim.openstreetmap.org/search?q=Curitiba&format=json&limit=5&addressdetails=1",
  { headers: { "User-Agent": "research-bot/1.0 (you@example.com)" } });
const places = await r.json(); // lat, lon, display_name, type, addressdetails
```

```python
import httpx
r = httpx.get("https://nominatim.openstreetmap.org/search", params={
    "q": "Curitiba", "format": "json", "limit": 5, "addressdetails": 1},
    headers={"User-Agent": "research-bot/1.0 (you@example.com)"}, timeout=15)
```

Research use: geocoding, place disambiguation, amenity inventories via the Overpass API (its query-language sibling — cafes, clinics, schools in any bounding box; no key, respect the 2-slot global quota).

## The wider shelf — quick reference

Same criteria (free, research-usable), less central. Format: what / endpoint / friction & notes.

### Encyclopedic & structured knowledge

| API | What it gives | Friction |
| --- | --- | --- |
| DBpedia SPARQL | The legacy linked-data extraction of Wikipedia | None; SPARQL; slower than Wikidata |
| DuckDuckGo Instant Answer | Topic cards + disambiguation for clean queries | None; instant answers only, **not** general search |
| Wikipedia (all languages) | Everything above in 300+ languages | None; swap the subdomain |
| Wikimedia REST feeds | On-this-day, featured articles, events | None; ⚠ rate-limits datacenter IPs |

### Academic & science

| API | What it gives | Friction |
| --- | --- | --- |
| DOAJ | 20k+ open-access journals, article search | None |
| CORE | 300M+ open-access papers aggregated | Works keyless for light use; free key for real volume |
| PubMed E-utilities | 36M biomedical citations | None; 3 req/s; free key raises |
| Europe PMC abstracts+fulltext | Open-access biomedical full text | None |
| GBIF | Species occurrence data (biodiversity) | None |
| USGS Earthquake Catalog | Seismic events, geojson | None |
| WHO GHO indicator API | Global health indicators | None; odata format |
| OpenAlex (repeated on purpose) | If you only call one academic API, it's this one | `mailto` polite pool |

### Government & statistics

| API | What it gives | Friction |
| --- | --- | --- |
| US Census (ACS) | US demographic tables | Free key (instant registration) |
| FRED | 800k+ US/Fed economic series | Free key (one-time registration) |
| BLS | US labor statistics timeseries | None at low volume; key raises; ⚠ blocks some datacenter IPs |
| Eurostat | EU statistics across all domains | None |
| data.gov CKAN | Catalog of ~300k US datasets | None; instance at catalog.data.gov |
| USAspending | Every US federal contract and grant | None |
| SEC EDGAR full-text | Filings search, 20+ years | None; descriptive UA mandatory |
| Congress.gov | US bills, amendments, members | Free key |
| CourtListener | US case law search | Free key; generous |
| REST Countries | Country facts in one call | None |
| WHO/OECD/IMF data APIs | Global econ/health series | Mixed: OECD and IMF have free endpoints; check per-series |

### Community, code & platforms

| API | What it gives | Friction |
| --- | --- | --- |
| Reddit (.json / OAuth) | Posts, comments, subreddits | OAuth now effectively required; ⚠ raw .json blocks datacenter IPs; respect rate limits or use the official API |
| npm registry search | Package metadata, downloads | None |
| PyPI JSON API | Python package metadata | None; `pypi.org/pypi/{name}/json` |
| Hugging Face Hub API | Models, datasets, spaces | None; free token raises limits |
| GitHub (rest + graphql) | Everything code-social | Token recommended |
| Launch Library 2 | Space launches and events | None |
| Open Food Facts | Product database (barcodes) | None |
| Frankfurter | FX reference rates (ECB) | None |

### News & monitoring

| API | What it gives | Friction |
| --- | --- | --- |
| GDELT DOC 2.0 | Global news search + timelines | None; bursty 429s — space calls |
| GDELT GKG | Global knowledge graph of themes/actors | None; heavy CSV payloads |
| RSS/Atom feeds (via feedparser) | Direct monitoring of chosen outlets | None — see `beyond-apis.md` |
| r.jina.ai | URL → clean markdown (reader proxy) | Free key; 401 without — see `beyond-apis.md` |

### Geodata & environment

| API | What it gives | Friction |
| --- | --- | --- |
| Open-Meteo (repeated on purpose) | Weather nowcast + historical | None; 10k/day |
| Nominatim / Overpass | Geocoding / raw OSM queries | 1 req/s; UA with contact; ⚠ datacenter-blocked |
| USGS/NASA APIs | Earth imagery, hazards, astronomy | Mixed: NASA APIs free key (DEMO_KEY works at low volume) |

## Failure modes and their meanings

| Symptom | Meaning | Response |
| --- | --- | --- |
| 429 with delay | Rate limit (yours or the shared pool's) | Exponential backoff; add the polite-pool param if one exists |
| 429 instantly, every time | Your IP class is pre-throttled (datacenter) | Route via user's machine, add credentials, or pick a sibling API |
| 403 on a known-free API | UA policy or IP block | Set a descriptive UA with contact; if it persists, ⚠ — document and route around |
| Timeout to a known-good host | Network-level block between environments | ⚠ — don't burn budget retrying; note it and move on |
| 400 with a helpful body | Your query, not the API | Read the error body — these APIs debug themselves better than blogs explain them |

One caveat the tables can't repeat often enough: **rate limits change faster than documentation**. When a call fails oddly, the endpoint's own error body (and `Retry-After` header) is the primary source — treat it accordingly.

