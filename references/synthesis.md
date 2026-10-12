# synthesis — from notes to report

> Merged from: `lingzhi227/agent-research-skills` (structured note format, synthesis-gates, report compilation) and `samber/cc-skills` deep-research (MIT — fact/synthesis separation, report-writer pattern). See ATTRIBUTION.md.

Synthesis is the conversion of many cited fragments into one defensible answer. It fails in two directions: too shallow (a link-dump wearing a conclusion) and too confident (an argument outrunning its evidence). The templates here prevent both.

## Structured notes (deep-read sources)

For every load-bearing source that gets a full read (5–15 in standard mode, more in deep), record one note block. A note that can't fill the limitations line wasn't a deep read:

```markdown
## [S12] Attention Is All You Need — Vaswani et al., 2017 (peer-reviewed venue: NeurIPS)
- **Problem:** sequence models were bottlenecked by recurrence; training couldn't parallelize across positions.
- **Contribution:** Transformer — self-attention replaces recurrence entirely.
- **Method:** scaled dot-product attention, multi-head; encoder-decoder, 8 heads.
- **Evidence:** BLEU gains on WMT14 EN-DE/EN-FR; ablations on head count.
- **Limitations:** quadratic memory in sequence length (authors note); English-centric benchmarks.
- **Connects to:** [S14] follow-up efficiency line; contradicts [S09] on recurrence necessity.
- **Quotes/numbers to cite:** BLEU 28.4 EN-DE; 3.5 days on 8 P100s.
```

Notes live in `{axis}.md` or a dedicated `notes.md`; the citation key (`S12`) links into the ledger. The **Connects to** line is where synthesis starts — cross-source relationships are the raw material of every synthesis move below.

Academic mode: notes carry the review status, and citation keys use `firstAuthorYearWord` (`[@vaswani2017attention]`), with BibTeX accumulating toward `references.bib` at the end.

## The synthesis moves

Run these in order; each produces report-ready material:

### 1. Taxonomy — fit the structure to the topic, not the template

Cluster findings into categories *the evidence suggests*, then name them. A generic "Overview / Details / Conclusion" is a failed taxonomy. A "three generations of approach, divided by what they assumed about memory cost" is a real one. If two clusters collapse into one, collapse them; if one cluster hides two distinct things, split it. The taxonomy is a finding, and it may be the most valuable one.

### 2. Comparison tables — build what the question implies

Every research type has an implied table: technical → approaches × criteria (performance, maturity, ecosystem, lock-in); competitive → competitors × dimensions (positioning, pricing, strengths, gaps); market → segments × size × growth × concentration. Table cells carry citations; cells that no source supports say "no data" — an empty-looking table with honest gaps beats a full-looking table with invented cells.

### 3. Pattern and gap analysis

- **Signal** — appears in ≥2 independent sources.
- **Single-sourcing** — appears once; reportable with a Low-confidence flag, never load-bearing.
- **Gap** — no source addresses it. Gaps go in a dedicated section: "What we could not determine" with why (paywalled data? no public measurement? niche too new?) and how to close it (what study, what access, what cost).

### 4. Fact/synthesis separation — the labeling discipline

Two voices in every report, always distinguishable:

- **Sourced fact:** "According to [S04], the market reached $4.2B in 2024."
- **Synthesis (yours):** "The pattern across three independent sources suggests the consolidation phase began earlier than the analysts' timelines imply."

A reader must be able to audit any sentence: is this the world talking, or the analyst? When synthesis appears inside a paragraph of sourced facts, mark the transition ("This suggests...", "Read together, these figures indicate..."). Recommendations rest on synthesis and *cite the facts that motivated them*.

## Report templates by type

Every report shares a skeleton — header, findings, recommendations, risks, next steps — and each type specializes the middle:

```markdown
# {Title}
> **Type:** market · **Date:** 2025-01-15 · **Depth:** standard
> **Assumptions:** scope=global, horizon=2024–2025 (state any scope inference here)
> **Methodology:** OpenAlex + GDELT + vendor docs; 14 sources, 6 deep-read; ledger: ./sources.md

## Executive summary        (≤200 words; the answer, not the topic)
## Key findings             (prose, 3–7 findings, each cited)
## {type-specific body}     (see below)
## Recommendations          (ranked by impact; evidence: [S-keys])
## Risks and uncertainties  (Low-confidence claims, unresolved conflicts, data gaps)
## What we could not determine
## Next steps               (follow-up research; decisions this enables)
## Sources                  (compiled from the ledger, with tiers)
```

Type-specific bodies:

- **market:** Sizing and growth → Segments → Competitive landscape → Pricing models → Regulatory headwinds
- **technical:** Options taxonomy → Comparison table → Benchmarks (with methodology critique) → Maturity/ecosystem per option → Decision guide
- **competitive:** Positioning → Product teardown → Pricing/packaging → Win/loss signals → Where they're vulnerable
- **academic:** Frontier (last 12–24 months) → Landscape (taxonomy) → Deep-read synthesis (per-cluster) → Open problems → Citation network notes (who cites whom; which clusters are dense vs sparse)
- **social-pulse:** Volume/trend over window → Key voices → Sentiment clusters → Notable disagreements → Engagement outliers (with the Low-tier flag)
- **trend:** Weak signals → Amplification conditions → Scenarios (2–4, with signposts that would confirm each) → What would falsify each scenario

## The report-writer pattern

When sub-agents are available, the final report is written by a **fresh sub-agent** that reads the notes and draft report but carries none of the gathering context. This is not bureaucracy — it's the cheapest available approximation of an outside editor. The writer gets: the original question, the notes directory, the draft report path, and this file's templates. The coordinator reviews its output against the pre-delivery gate (see `evaluation.md`) before shipping.

## Quality gate (before delivery)

- [ ] Executive summary answers the question, not the topic
- [ ] Every finding cites; every critical finding is High confidence or flagged
- [ ] Tables have no invented cells; gaps are explicit
- [ ] Fact/synthesis separation survives a spot-check of 5 random paragraphs
- [ ] Recommendations trace to evidence; low-evidence recommendations say so
- [ ] Risks section includes every unresolved conflict from the ledger
- [ ] Volatile-topic freshness note present ("snapshot as of {date}; re-run within 30 days")
- [ ] The deep-read mandate was met (5–15 sources actually read, notes recorded) — or the report explicitly labels itself snippet-based

## PDF export (optional, when offered)

```bash
pandoc report.md -o report.pdf --pdf-engine=weasyprint   # or wkhtmltopdf, or LaTeX
md-to-pdf report.md                                      # node fallback
```
