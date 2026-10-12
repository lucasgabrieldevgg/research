# method — the research loop

> Merged from: `samber/cc-skills` deep-research (MIT — research types, confidence ladder, fan-out, critical rules), `lingzhi227/agent-research-skills` deep-research + research-planning (phase gates, source-quality ladder, deep-read mandate), `mvanhorn/last30days-skill` (recency windows, normalize-filter-score-dedupe pipeline). See ATTRIBUTION.md.

Research is not search. Search retrieves pages; research converts them into a verified, cited, decision-ready picture of a topic. The difference is process: framing before fetching, evaluation before believing, synthesis before writing. This file is that process.

**Persona check:** operate as a senior research analyst — skeptical of single sources, obsessed with citations, allergic to papering over uncertainty. If the environment has no web access of any kind (no search, no fetch), halt and say so; do not synthesize from memory and present it as research.

## The loop at a glance

```
Frame → Plan → Gather → Verify → Synthesize → Deliver
  ↑__________________________________________|
        (gaps found late loop back to Gather, once)
```

Every stage writes to disk immediately. Nothing is batched for the end. The output lives in a dual structure:

```
research/
├── 2025-01-15-{type}-{topic}.md      # the report (deliverable)
└── 2025-01-15-{type}-{topic}/        # per-axis notes + source-ledger.jsonl
    ├── sources.md                     # the source ledger (running)
    ├── {axis-1}.md                    # one file per research axis
    ├── {axis-2}.md
    └── ...
```

Name the report `{date}-{type}-{topic}.md`, lowercase-hyphenated, topic portion under 50 chars. Check `./research/` for an existing report on the topic first — if one exists, ask: extend, update, or start fresh?

## Stage 0 — Frame

Classify before you search. Two questions: **what kind of research is this, and how deep does it go?**

### The 12 research types

| Type | Question it answers | Typical deliverable |
| --- | --- | --- |
| `market` | Who buys, who sells, at what price, how fast is it growing? | Sizing + segments + trends report |
| `domain` | How does this industry/field actually work? | Structure + ecosystem + regulation overview |
| `technical` | Which approach/tool/architecture wins, and why? | Trade-off analysis + benchmark comparison |
| `competitive` | What is competitor X doing and how do they win/lose? | Positioning teardown |
| `product` | What is product X, in detail? | Feature/UX/roadmap teardown |
| `academic` | What does the literature say about X? | Literature survey with citation network |
| `person/org` | Who is this person/company, really? | Due-diligence brief |
| `financial` | Who funded whom, at what valuation? | Funding landscape |
| `legal` | What patents/cases/rules govern this space? | IP/regulatory landscape |
| `trend` | What is emerging that others miss? | Signal scan + scenarios |
| `community` | What does the community around X think/say/need? | Sentiment + key-voices map |
| `social-pulse` | What are people saying about X *right now*? | Recency-windowed sentiment (last 7/30 days) |

If none fit, infer the closest type and design your own axes — the loop is the same regardless of type.

### Depth modes

| Depth | When | Behavior |
| --- | --- | --- |
| **Quick** | Narrow, time-sensitive; user says "brief" or "quick" | Auto-scope, single gather pass, deliver |
| **Standard** | Default | Full loop, 5–10 axes of gathering |
| **Deep** | "Thorough", "exhaustive", critical decision at stake | Full loop + outline-refinement pass + critique pass before delivery |

### Scope interview — only when needed

If the prompt is specific (topic, type, and goals are clear): **skip the interview**. State assumptions in the report header instead:

> **Assumptions:** type=market, scope=global, horizon=2024–2025, goals=TAM sizing and growth drivers.

If the prompt is vague ("research blockchain", "tell me about AI"), ask three questions before anything else: (1) what type, (2) what specific questions should this answer, (3) any geographic/time/segment constraints. One question at a time, 2–4 options each, through the environment's question tool — never as a wall of prose.

Also capture the **recency window** here (does the user care about the last 30 days, the last 2 years, or all time?) and the **volatility** of the topic (pricing and regulatory facts rot in weeks; textbook physics does not).

## Stage 1 — Plan

Turn the frame into an axis plan. Axes are independent slices of the question — each one becomes one gather task (and, if sub-agents are available, one sub-agent).

**How many axes:** 3 for quick, 5–10 for standard, up to 20 for deep. Examples for `market`: sizing, growth drivers, segments, pricing models, competitive landscape, distribution channels, regulatory headwinds.

**Source strategy per axis.** Decide *up front* which parts of the toolbox (see `free-apis.md`, `beyond-apis.md`, `paid-apis.md`) serve each axis. The selection matrix:

| Question shape | First-line sources |
| --- | --- |
| Definitions, established facts, entities | Wikipedia REST, Wikidata |
| Academic state of the art | OpenAlex, arXiv, Semantic Scholar, Crossref |
| Biomedical/health | Europe PMC, PubMed |
| What happened recently / news coverage | GDELT, RSS feeds of key outlets |
| Practitioner sentiment, tool debates | HN Algolia, Stack Exchange, Reddit (via MCP or scraping), X |
| Code/ecosystem maturity | GitHub API, npm/PyPI |
| Market/economic numbers | World Bank, FRED, Eurostat, BLS, Census |
| Companies/filings | SEC EDGAR, OpenCorporates, CourtListener |
| Books/long-form background | Open Library, Internet Archive, Gutendex |
| Numbers nobody publishes as data | General web search (paid tier if available) |

**Budget:** set a time budget and a cost budget before gathering. Cost is zero unless paid APIs are in play — if they are, write the expected spend down (e.g., "≤500 Tavily credits") and track against it.

**Write the plan to the notes directory as `plan.md`** — axes, sources per axis, budget, recency window. This takes two minutes and prevents the classic failure mode: gathering everything about everything.

## Stage 2 — Gather

Fan out. If the environment supports sub-agents, spawn them in a single message — one per axis, each with a self-contained prompt (topic, axis, goals, constraints, citation rules). If not, work the axes sequentially, in priority order.

Each axis's gather pass:

1. **Query the toolbox** for that axis (the API calls themselves — see the integration files).
2. **Fetch the actual sources** it returns. A search result is a lead, not a finding — the page behind the URL is the finding.
3. **Write findings to `{axis}.md` as prose paragraphs with inline citations** — not bullet dumps. "The market reached $4.2B in 2024 [Source]" beats "* Market: $4.2B". Bullets are for true enumerated lists only.
4. **Append every source used to the ledger** (see `evaluation.md` for the schema) — URL, accessed date, tier, one-line claim it supports.

Rules that keep gathering honest:

- **Every claim cites a source URL.** Unsourced assertions are guesses wearing findings' clothes.
- **Write-as-you-go.** After each axis completes, its file is written. Never batch.
- **Stop conditions:** an axis is done when its questions are answered with ≥2 independent sources, or when 15 minutes of increasingly-creative queries yield nothing new. Write "No sources found for X" in that case — an honest gap, not a blank.
- **Recency discipline:** for volatile topics, prefer sources inside the window; for every claim older than the window, check whether a newer source contradicts it.

### Phase gates (standard and deep modes)

Before synthesis, verify the artifacts exist on disk — missing files mean the phase didn't happen, no matter what the transcript says:

| Gate | Check |
| --- | --- |
| Gather → Verify | Every planned axis has a `{axis}.md` with ≥1 cited finding, or an explicit "No sources found" |
| Verify → Synthesize | `sources.md` ledger exists; every critical claim has ≥2 ledger entries or a `confidence: Low` tag |
| Synthesize → Deliver | Taxonomy/comparison content exists; gaps section drafted; every section cites ≥1 deep-read source |

### The dedupe pipeline (multi-source gathering)

When several sources return overlapping material (typical when running the social-pulse type), normalize → filter → score → dedupe before writing:

- **Normalize**: same story from 5 outlets becomes one item with 5 citations.
- **Filter**: drop items outside the recency window or below the engagement floor.
- **Score**: engagement (upvotes/comments) × source tier × recency — a ranking signal, never a truth signal. High engagement on a Low-tier source is still a Low-tier claim.
- **Dedupe**: by canonical URL and by content similarity, keeping the highest-tier citation set.

## Stage 3 — Verify

Evaluation happens per-claim, not per-source. The full framework lives in `evaluation.md`; the operating summary:

- **Tier the source** (Primary / Established / Low) at ledger time.
- **Triangulate critical claims** — numbers, market sizes, growth rates, competitive positioning, anything a decision will rest on — across **2+ independent sources**, or tag them `confidence: Low`. Independent means different origina (two outlets both citing one press release are one source).
- **Flag conflicts explicitly.** "Source A reports 4.2B; Source B reports 3.8B; the delta traces to different segment definitions" — never silently pick one.
- **Confidence ladder:** High = 2+ reputable independent sources agree. Medium = 1 reputable source. Low = blogs, forums, single opinions, or inference.

## Stage 4 — Synthesize

This stage is where research-quality lives or dies.

**The deep-read mandate:** for standard and deep modes, the 5–15 most load-bearing sources must be *read* — full text or substantial sections — not summarized from abstracts and snippets. Abstract-only synthesis is the research equivalent of reviewing a book by its cover. Record structured notes for each (template in `synthesis.md`).

**Then the synthesis moves**, in order:

1. **Taxonomy** — cluster findings into a structure that fits *this* topic, not a generic template.
2. **Comparison** — build the tables that the question implies (approaches × criteria, competitors × dimensions, tools × trade-offs).
3. **Pattern and gap analysis** — what repeats across independent sources (signal), what appears exactly once (unverified), what no source addresses (gap).
4. **Fact/synthesis separation** — "According to [Source], X" for findings; "This suggests Y" / "The pattern indicates Y" for your analysis. Never present inference as fact, and never bury synthesis inside sourced paragraphs where the reader can't tell which is which.
5. **Outline refinement (deep mode only)** — before writing, check: did findings contradict the framing? did an unplanned angle emerge? are any sections evidence-starved? If yes, restructure and run 2–3 targeted gap-fill queries (time-boxed to 5 minutes).

**Critique pass (deep mode only):** red-team the synthesis before delivery. What's missing? What could be wrong? What alternative explanation fits the same data? If a critical gap emerges, loop back to Gather — **once**. One loop, then deliver with the gap documented.

## Stage 5 — Deliver

Reports follow the type-appropriate template in `synthesis.md`. Every report carries, without exception:

- **Header block:** topic, type, date, scope assumptions, methodology note (what was searched where).
- **Key findings** as prose, each tied to sources.
- **Recommendations/answers** ranked by impact, each with its evidence base.
- **Risks and uncertainties:** data gaps, Low-confidence claims, unresolved conflicts.
- **Next steps:** what follow-up research would change the answer.

Volatile topics (pricing, regulatory, competitive) get a freshness note: "Snapshot as of {date}; re-run within 30 days before acting." Offer PDF export if the environment has pandoc or md-to-pdf available.

## Hard rules

These are non-negotiable across all modes — they're what separates research from content generation:

1. **Every claim carries a citation.** No exceptions for convenience.
2. **Critical claims need 2+ independent sources** or an explicit `confidence: Low`.
3. **No fabrication.** If a source doesn't exist, say so. "No sources found for X" is a valid finding.
4. **No silent conflict resolution.** Disagreement between sources is reported, not adjudicated quietly.
5. **Write-as-you-go.** Disk after every axis, every stage. No end-of-run batching.
6. **Abstracts are not reading.** Load-bearing sources get read; synthesis built on snippets gets flagged as such.
7. **No inference presented as fact.** "This suggests..." — always labeled.
8. **No diving without framing.** An ambiguous topic gets the scope interview first.

## Degradation paths

- **No sub-agents:** run axes sequentially in priority order; everything else holds.
- **No paid APIs:** the free toolbox covers encyclopedic, academic, news, community, government, and code categories. General web search is the only gap — if the environment has WebSearch, use it as the general tier.
- **No web access at all:** halt and say so. Do not fake research from training memory; offer a knowledge-based briefing clearly labeled as such instead.
