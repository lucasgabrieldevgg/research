# scholar

A big skill for **research management** — running rigorous multi-source research and wiring research data into projects. Born from merging the best research-management skills found on GitHub/ClawHub with a live-verified catalog of free research APIs.

```
Research is not search. Search retrieves pages;
research converts them into a verified, cited, decision-ready picture.
```

## What's inside

| Piece | What it does |
| --- | --- |
| **The research loop** (`references/method.md`) | 6-stage process: Frame → Plan → Gather → Verify → Synthesize → Deliver. 12 research types, 3 depth modes, phase gates, hard rules |
| **Source discipline** (`references/evaluation.md`) | Tier ladder (Primary/Established/Low), confidence ladder, triangulation and conflict protocols, the source ledger |
| **Synthesis craft** (`references/synthesis.md`) | Structured notes, taxonomy/comparison/gap moves, fact-vs-synthesis separation, report templates per research type |
| **Free API toolbox** (`references/free-apis.md`) | 15 deep-dives (Wikipedia, Wikidata, OpenAlex, arXiv, Crossref, Semantic Scholar, Europe PMC, PubMed, GDELT, HN, Stack Exchange, GitHub, Open Library, World Bank, Open-Meteo, Nominatim) + ~40-entry quick-reference — all live-verified, TS + Python snippets |
| **Paid shortlist** (`references/paid-apis.md`) | The 6 worth paying for — Brave Search, Tavily, Exa, SerpAPI, Serper, Perplexity Sonar — with a when-to-pay decision tree and cost discipline |
| **Beyond APIs** (`references/beyond-apis.md`) | MCP servers (verified packages + config), bulk datasets (HF, Common Crawl, Internet Archive), polite scraping (trafilatura), RSS/Atom monitoring (feedparser) |
| **Integration patterns** (`references/integration.md`) | Env contract, zero-dep clients, retry/backoff, query cache, free-first provider router, cost meter, test fixtures |

Plus a Cursor rules export at `exports/scholar.mdc`.

## Install

**Claude Code / GLM / any SKILL.md host:**
```bash
git clone https://github.com/lucasgabrieldevgg/scholar.git ~/.claude/skills/scholar
```

**Cursor:** copy `exports/scholar.mdc` to `.cursor/rules/scholar.mdc`.

**ClawHub / skills.sh:** the folder is standard skill layout — publish as-is.

## Method (the 70%)

The loop and its hard rules are the skill's center of mass: framing before fetching (12 research types, depth modes, recency windows), planning axes with a source strategy per axis, gathering with write-as-you-go discipline, verifying with tier/confidence ladders and explicit conflict handling, synthesizing with a deep-read mandate (abstracts are not reading), and delivering reports that state their gaps. The API catalog exists to serve the loop, not the other way around.

## Verification

Every free-API endpoint in `references/free-apis.md` was live-tested (status, latency, response shape) in January 2026. Entries marked ⚠ were unreachable from the verification sandbox's datacenter IP but are established free services — the failure-modes table explains how to read each symptom. MCP packages were verified against npm/PyPI with exact versions. Paid pricing is marked indicative — vendors change it faster than files track.

## Attribution

`scholar` is an original synthesis (MIT) merging upstream material from `samber/cc-skills` (MIT), `lingzhi227/agent-research-skills`, and `mvanhorn/last30days-skill` (MIT), plus original live API verification. See [`ATTRIBUTION.md`](ATTRIBUTION.md).

## License

MIT for this repo's original content. Upstream material retains its own licenses — see ATTRIBUTION.md.
