# Attribution & Sources

`research` is an original synthesis commissioned for this repo: the research-method core was merged from three upstream skills, and the entire API/MCP/tooling catalog was verified live (January 2026) for this project — endpoint reachability, response shapes, package versions, and pricing-page readings. The synthesis, structure, and merged rule set are MIT. Upstream material retains its original license.

## Upstream skills (merged)

| Source | What was merged | License |
| --- | --- | --- |
| `samber/cc-skills` — skills/deep-research (v1.3.0, by Maxme Courant & Samuel Berthe) | The 12-research-type taxonomy, depth modes (quick/standard/deep), parallel fan-out orchestration, the tier ladder (Primary/Established/Low) and confidence ladder (High/Medium/Low), the critical rules (cite every claim, 2+ sources for critical claims, write-as-you-go, prose-first findings, admit gaps), the conflict protocol, the dual output structure (report + notes directory), degradation paths | MIT |
| `lingzhi227/agent-research-skills` — deep-research, research-planning, citation-management, literature-search | Phase gates with on-disk verification, the academic peer-review sub-ladder with `(preprint)` marking, the deep-read mandate (abstracts are not reading), source-count targets, structured note format with connections, the harvest/validate/dedupe citation lifecycle | No LICENSE file in the upstream repo — public-source skill collection; credited here as merged source. If the author specifies a license, this table updates |
| `mvanhorn/last30days-skill` (v3.28.0, by Matt Van Horn) | Recency-window research, the normalize → filter → score → dedupe pipeline for overlapping multi-source results, engagement-as-signal (never as truth), the source-health check concept, multi-platform social gathering patterns | MIT |
| `Imbad0202/academic-research-skills` (by Cheng-I Wu) | Architectural reference only (multi-agent pipeline shape: architect → screening → synthesis → compiler); no text merged | CC (see upstream repo) |

## Original verification work

- **Upstream landscape survey**: 5 candidate repositories cloned and studied before synthesis (GitHub + ClawHub search, January 2026). `Orchestra-Research/AI-Research-SKILLs` (MIT) was surveyed and excluded by scope decision — it targets AI/ML-engineering research (fine-tuning, distributed training, inference), not multi-source web research; no content merged.
- **Live API verification**: ~40 free-API endpoints tested (HTTP status, latency, JSON response shape) from this project's sandbox in January 2026; datacenter-IP blocks documented as ⚠ with failure-mode semantics. Results in the repo's research notes, reflected throughout `references/free-apis.md`.
- **MCP package verification**: npm/PyPI registry checks with exact versions for the 11 servers listed in `references/beyond-apis.md`.
- **Paid pricing readings**: vendor pricing pages fetched January 2026 (SerpAPI, Tavily reachable; others documented as indicative with verify-current guidance).
- **Integration patterns, cost-meter pattern, fixture strategy, method-stage wording**: original to this repo.

## Individual credits

- **Samuel Berthe** and **Maxme Courant** — the deep-research skill that forms this skill's methodological backbone.
- **lingzhi227** — the academic research skill collection (phase gates, deep-read mandate, citation lifecycle).
- **Matt Van Horn** — the last30days social-research engine (recency windows, dedupe pipeline).
- The API providers themselves — Wikimedia, OpenAlex, arXiv, Crossref, Semantic Scholar, Europe PMC, NCBI, GDELT, Algolia (HN), Stack Exchange, GitHub, Open Library/Internet Archive, World Bank, Open-Meteo, OpenStreetMap — whose free, well-documented endpoints made the toolbox possible.

Any errors in the merged synthesis are this repo's, not theirs.

If you redistribute `research`, keep this file and the per-file credits with it.
