# evaluation — source discipline

> Merged from: `samber/cc-skills` deep-research (MIT — tier ladder, confidence ladder, conflict rules) and `lingzhi227/agent-research-skills` (peer-review priority ladder). See ATTRIBUTION.md.

A source's job is to be trustworthy *for a specific claim*. The same arXiv preprint is Primary-tier for "what the authors propose" and nearly worthless for "this method works". Evaluate per-claim, and make the evaluation visible in the ledger.

## The tier ladder

| Tier | What qualifies | Examples |
| --- | --- | --- |
| **Primary** | Produced by the entity with first-hand knowledge; no intermediary | Official docs, government filings (SEC EDGAR, FRED), peer-reviewed papers, court records, the vendor's own pricing page, raw data from the API itself |
| **Established** | Editorial process, reputation at stake, professional accountability | Major publications, analyst firms, textbook publishers, standards bodies |
| **Low** | No editorial layer; anyone can publish anything | Blogs, forums, Reddit/HN comments, single tweets, SEO content farms, AI-generated summaries |

Two rules make the ladder real:

1. **Tag at ledger time, not at write time.** The tier travels with the source into the ledger (below), so synthesis can't accidentally promote a blog to fact status.
2. **Low-tier is allowed, but visible.** Community sources are often the *only* signal for practitioner sentiment. Use them, flag them: "according to a highly-upvoted Hacker News thread (Low tier)".

### The academic sub-ladder (academic-type research only)

For literature work, tier within Primary by review status — peer-reviewed material outranks preprints:

1. Top peer-reviewed venues (NeurIPS, ICLR, Nature, JMLR, TACL...)
2. Peer-reviewed journals generally
3. Workshop papers (reviewed, lower bar)
4. Highly-cited arXiv preprints (likely quality, unverified status)
5. Recent arXiv preprints — use cautiously, mark `(preprint)` in every citation

Preprints are supplementary evidence, or the only evidence for results <3 months old. A citation to a preprint that exists in peer-reviewed form cites the reviewed version.

### The AI-slop caveat (applies to every tier)

Search results increasingly surface AI-generated summaries of AI-generated summaries. Tells: no original reporting, no primary links, hedged-everything prose, a "last updated" date but no author. A page that cites its own sources can be treated as its sources' tier (go read those). A page that cites nothing is Low tier regardless of how polished it looks — polish is cheap now.

## The confidence ladder

Confidence attaches to **claims**, combining tier and independence:

- **High** — 2+ independent reputable sources agree. *Independent = different origina.* Two articles rewriting one press release are one source. Two datasets from the same government agency are one source.
- **Medium** — exactly 1 reputable source (Primary or Established).
- **Low** — Low-tier sources only, or inference, or estimates extrapolated by you.

Claims that will carry decisions (market size, growth rate, pricing, "X supports Y", legal status) start at Low until triangulated. Numbers deserve particular suspicion: check whether competing figures trace to different definitions (revenue vs GMV; calendar vs fiscal year) before calling it a conflict.

## Triangulation protocol

For each critical claim:

1. Find two sources that state it **independently** (check the origina: does B link to/cite A? Then B is not independent of A).
2. Prefer sources of **different kinds** (one filing + one analyst report beats two analyst reports).
3. If they disagree, run the conflict protocol below — agreement you manufactured by ignoring disagreement is the worst outcome in research.

## Conflict protocol

When sources disagree on a material claim:

1. **Report both** with citations: "Source A reports X; Source B reports Y."
2. **Diagnose** if cheap: different definitions, different dates, different measurement methods, different populations? Say which fits the question at hand.
3. **Never silently pick** a winner. If forced to operate on one number, state the choice, the reason, and the other candidate: "We use B's figure because it matches the segment definition in scope; A's broader definition would inflate it."
4. If the conflict survives diagnosis, it lands in the report's *Risks and Uncertainties* section.

## The source ledger

Every gathered source lands in `sources.md` (human-readable) backed by `source-ledger.jsonl` (machine). One JSON line per source:

```json
{
  "id": "s17",
  "url": "https://api.worldbank.org/...",
  "title": "World Bank — Brazil GDP (NY.GDP.MKTP.CD)",
  "accessed": "2025-01-15",
  "tier": "Primary",
  "kind": "api+docs",
  "supports": ["GDP figure in sizing section"],
  "confidence": "High",
  "notes": "Retrieved directly via API; no intermediary"
}
```

Ledger hygiene rules:

- **Accessed date is mandatory** — it's what makes a recency claim auditable later.
- **API endpoints are first-class sources.** Data pulled directly from World Bank's API is a Primary source; cite the endpoint and the query, not "the World Bank website".
- **The ledger is append-only.** Corrections append a new line; the history is evidence.
- **Every axis file's citations resolve to ledger entries.** The report's bibliography compiles from the ledger, so nothing appears in the bibliography that wasn't actually used, and vice versa.

## Citation format (in report prose)

```
[Source Name](url) (accessed YYYY-MM-DD, confidence: High|Medium|Low, tier: Primary|Established|Low)
```

Academic mode adds the review status: `[@vaswani2017attention]` with `(preprint)` markers for unreviewed entries, full BibTeX emitted at the end (see `synthesis.md`).

## Recency and volatility

Before trusting any time-sensitive claim, ask:

- **Is the fact perishable?** Pricing, availability, headcount, regulatory status, API rate limits — treat as stale after ~30 days. Definitions, history, physics — durable.
- **Is the source's date real?** A "last updated 2024" footer on a page whose data is from 2019 is common. Find the date *of the data*, not the page.
- **Does a newer source contradict it?** For volatile topics, every out-of-window claim gets one freshness check: search for a newer statement; if found, both go in the ledger and the conflict protocol runs.

## Pre-delivery gate

Before any report ships, one final pass over the ledger:

- [ ] Every claim in the report maps to ≥1 ledger entry
- [ ] Every decision-critical claim is High confidence, or explicitly flagged Low with rationale
- [ ] Every conflict found during research appears in the report (not resolved silently in the ledger)
- [ ] Every citation in the report resolves to a live, correctly-attributed ledger URL
- [ ] No source appears in the bibliography without a corresponding claim in the body
