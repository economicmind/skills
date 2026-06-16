---
name: bottom-up-market-size
description: >
  Size a market bottom-up from an Economic Mind market report and stress-test that number against
  independent sizing approaches - the headline size, the transparent per-company build (each player's
  revenue, share-in-market, in-market revenue, confidence and justification carried verbatim from the
  report), an explicit read on how much to trust the headline and why, and a football-field comparison
  of the bottom-up figure against 2-3 credible cross-checks. Use whenever the user wants to "size the X
  market bottom-up", "how big is X and how sure are we", "bottom-up market size", "build / show the
  market-size buildup", "triangulate the size of X", or "benchmark our market size against other
  estimates". The bottom-up figures come ONLY from the Economic Mind report, unchanged; cross-checks come
  from a credible-source research pass. The deliverable is data (Markdown + JSON); presentation is left to
  the client.
---

# Bottom-up market size

Size a market the way a careful analyst would: take Economic Mind's **bottom-up build** as the spine, show
every company that rolls up into it, say plainly how much the headline can be trusted and why, then put
that number on a **football field** next to a few independent estimates so the reader can judge it. The
deliverable is **data** - a Markdown report plus a JSON sidecar - not a styled document; this skill is
about the research and the numbers, and layout is the client's job.

The whole value is **credibility under scrutiny**: a market size is only as good as the build beneath it
and the cross-checks beside it. So nothing is hand-waved - the build is carried verbatim from the report,
the confidence is reasoned from the report's own stamps, and every cross-check names a credible source.

## Two source channels, never mixed

1. **The bottom-up build - ONLY the Economic Mind report, unchanged.** The headline size, the per-company
   revenue, share-in-market, in-market revenue, confidence ratings, justifications, the scope boundary and
   the vintage all come **straight from the report**. Do **not** recompute, re-estimate, re-rank, or
   "improve" any of them, and do not pull a player's revenue from elsewhere. You are presenting the
   report's build faithfully, not rebuilding it. (The only arithmetic permitted on it is summing the
   in-market revenues to show the build-to-total and reconciling that to the report's stated headline -
   shown transparently, never overriding the report.)
2. **The cross-checks - a credible-source research pass.** The alternative sizing approaches on the
   football field are **not** in the report; you construct 2-3 of them from independent credible sources
   (see the source rules). These sit **beside** the bottom-up for comparison and **never alter it**. This
   is the one place the web is used, and only for the cross-checks.

Keeping these apart is what makes the exercise honest: the anchor stays pristine and the triangulation is
clearly a separate, independently-sourced opinion.

## Work to files first, then read them back

Context can be compressed mid-task, and a market build has a lot of cells. So **immediately after each
tool call returns, write its raw output to a file** before the next call - the report content, then each
cross-check as you research it. Build the deliverable by reading the files back, not from recall. Create
one working directory at the start, e.g. `em-size/`.

## Source rules for the cross-checks (strict)

The football field is only as credible as its weakest source, so the bar is high. Use **only**:
(a) primary filings & registries (Proff, Brønnøysund, Allabolag, S&P Capital IQ, Creditsafe, audited annual
reports); (b) official statistics (Eurostat, SSB, SCB, national statistics offices); (c) regulators &
standards bodies (IMO, USCG, national building authorities); (d) listed-company reports and reputable
financial data; (e) named industry associations.

**Never** use market-research aggregator / press-release "market-size" mills - MarketsandMarkets, Grand
View, IMARC, Verified Market Reports, SNS Insider, Precedence, FactMR, Custom Market Insights, SkyQuest,
Fortune Business Insights, Mordor, Spherical and the like. Their figures are unreliable and
scope-inconsistent. When web-searching, pass these as `blocked_domains`. Build cross-checks from **first
principles on credible bases** (e.g. official installation statistics × a fire-protection share; a listed
group's disclosed segment revenue grossed for the tail; installed base × annual value). If a number exists
only from a mill, do not use it - say no credible independent estimate was available and lean on the build.

**Restate scope.** Any external estimate almost certainly uses a different market definition. Before it can
sit on the football field, restate it to the **report's in-scope definition** (geography, product boundary,
revenue basis) and say how you did so in its comment. Never fabricate; if you cannot scope-adjust an
estimate honestly, drop it.

## Workflow

### Phase 1 - Resolve the market & pull the build
1. `mkdir -p em-size`.
2. `search_markets("<the user's market phrase>")` (limit ~5). If the top hit's `score` is ≥1.5× the
   second's (or there is one hit), use it and announce the choice. Otherwise show the top 3 (Market,
   Geography, Size, Relevance) and ask the user to pick. If nothing matches, say the market is not covered
   and stop - do not improvise a size.
3. `get_market(slug)` for the catalog, then `get_market(slug, "bottom-up-supply-side")` (use the catalog's
   actual element name) to get the definition, headline size, and the per-company build. Tables come back
   as markdown - read them, don't guess.
4. **Immediately write** the raw element to `em-size/report-bottomup.md`, and write `em-size/build.csv` -
   one row per company, every field carried verbatim from the report (leave blank only where the report
   itself is blank):
   ```
   rank,company,hq,total_revenue,basis,share_in_market_pct,in_market_revenue,confidence,revenue_source,justification
   1,Co A,NO,402.7,reported,38%,153.0,H,Proff (NO),"Disclosed FY24 revenue; share set by product-line split"
   2,Co B,SE,210.4,estimated,55%,115.7,M,S&P Capital IQ,"Revenue estimated from filings; share from end-market mix"
   ...
   ```
   `basis` = reported vs estimated of the **total revenue**; `confidence` = the report's confidence on the
   **share in market** (the scope share, not a market share). Keep both - they mean different things.
5. Also capture, verbatim, into `em-size/headline.txt`: the report's **stated headline size**, its
   **vintage** ("2025 or latest available"), its **methodology note** (e.g. "top-N in-market revenue; no
   long-tail estimate"), the **in-/out-of-scope boundary**, any **concentration** figures (top-2/5/10
   shares), and the **player count**.

### Phase 2 - Build the cross-checks (credible research pass)
Construct **2-3 independent** sizing approaches to triangulate the headline, following the source rules
above. Typical approaches: a top-down (official statistic × a credible share), an installed-base × annual
value, and one independent third-party/analyst range (credible only). For each, do the arithmetic from
first principles, restate it to the report's in-scope definition, and record it - **immediately after the
research** - in `em-size/crosschecks.csv`:
```
method,approach,low,point,high,confidence,scope_note,source
top_down_capex,"Official construction stats × fire-protection share",480,560,640,M,"Restated to Nordic, OEM revenue basis","Eurostat 2024; SSB"
listed_grossup,"Listed groups' segment revenue grossed for tail",500,590,690,M,"In-scope geography only","Company annual reports 2024"
analyst_range,"Independent analyst estimate (scope-adjusted)",430,520,610,L,"Broader-scope figure restated down","<named credible source>"
```
If a genuine research pass yields nothing credible (only mills), record that fact and present the bottom-up
alone - do not invent a cross-check to fill the field.

### Phase 3 - Verification (do not skip)
Work from the files.
1. `cat em-size/build.csv em-size/headline.txt em-size/crosschecks.csv`.
2. **Fidelity check (the important one):** confirm every figure in `build.csv` and `headline.txt` matches
   the report exactly - same numbers, same confidence stamps, same scope wording. You changed nothing.
3. **Build-to-total reconciliation:** sum `in_market_revenue` across companies and compare to the report's
   stated headline. They should match (bottom-up = sum of in-market revenues). If they differ (e.g. the
   report adds a long-tail estimate or rounds), **show both and explain the difference** - never silently
   adjust either. State the formula: `Σ in-market revenue = … vs report headline = …`.
4. **Cross-check integrity:** each cross-check names a credible source, was scope-restated to the report's
   definition, and uses no mill. Drop any that fails.

### Phase 4 - Confidence read
Communicate how much to trust the headline **and why**, grounded in the report's own stamps - not a vibe.
Carry the report's overall confidence/caveat verbatim, then add a transparent **coverage summary computed
from the per-row stamps** (clearly labelled as derived from the report's own data, not new estimation):
- share of in-market revenue resting on **reported** vs **estimated** total revenue (`basis`);
- share of in-market revenue carrying **high vs low confidence on the scope share** (`confidence`);
- concentration (how much of the size sits in the top few names - a few large estimated lines can swing it);
- the standing caveats: vintage ("latest available", not one clean year), geography/Nordic emphasis, and
  "no long-tail" if the report says so.
Write `em-size/confidence.txt`. The point is a reader should see exactly which parts of the number are
solid and which are soft.

### Phase 5 - Assemble the deliverable (data; the client styles it)
Build the Markdown and JSON **from the files**; do not re-key a number from memory. Markdown structure:
```
# Bottom-up market size - <market> (<currency>, <vintage>)
*From the Economic Mind "<market>" bottom-up report (figures carried verbatim). Cross-checks independently
researched from credible sources. AI-assisted - confirm against the report and the cited sources before
relying on it.*

## Headline
- **Market size:** <report headline> (<vintage>)
- **Basis:** <report methodology note, e.g. top-N in-market revenue; no long-tail>
- **Scope:** <in-/out-of-scope boundary, verbatim>
- **Players covered:** N  ·  **Concentration:** top-5 = …% (if stated)
- **Confidence:** <overall, with the one-line why from the confidence read>

## Bottom-up build (per company, from the report)
| Rank | Company | HQ | Total revenue | Basis | Share in market | In-market revenue | Conf. | Justification |
|-----:|---------|----|--------------:|-------|----------------:|------------------:|:-----:|---------------|
| 1 | Co A | NO | 402.7 (reported) | … | 38% | 153.0 | H | Disclosed FY24 … |
*All figures carried verbatim from the Economic Mind report. Basis = reported/estimated revenue;
Conf. = confidence on the share in market.*
Σ in-market revenue = <sum>  ·  report headline = <headline>  (<match / difference explained>)

## Confidence & assumptions
- Overall: <report's confidence/caveat, verbatim>.
- Coverage (computed from the report's own stamps): <X>% of the size rests on reported revenue, <Y>% on
  estimated; <Z>% carries high confidence on the scope share. Top-<k> names = <…>% of the size.
- Key assumptions: <scope-share basis, geography, vintage, no-long-tail>.

## Triangulation (football field)
| Method | Approach | Low | Point | High | Conf. | Source & scope note |
|--------|----------|----:|------:|-----:|:-----:|---------------------|
| Bottom-up (anchor) | EM report build | - | <headline> | - | <conf> | Economic Mind "<market>" report |
| Top-down | … | 480 | 560 | 640 | M | Eurostat 2024 … |
| Listed gross-up | … | 500 | 590 | 690 | M | Annual reports 2024 |
**Converged view:** <range>, with the bottom-up at <headline>. <Does the anchor sit inside the
cross-check range? If it's an outlier, say why - scope, vintage, or tail.>

## Sources & method
- **Bottom-up** - Economic Mind "<market>" report (bottom-up supply-side); per-company revenue providers
  as stated (e.g. Proff, S&P Capital IQ). Carried verbatim; not recomputed.
- **Cross-checks** - <each method's named credible source>; scope-restated to the report's definition.
- **Excluded:** market-research "mills" (unreliable / scope-inconsistent). **Caveats:** <vintage,
  geography, long-tail>.
```

JSON sidecar (`em-size/market-size.json`) - machine-readable mirror:
```json
{
  "market": "Nordic fire protection", "currency": "EUR", "vintage": "2025 or latest available",
  "headline": {"size": 590.0, "basis": "top-N in-market revenue; no long-tail",
               "scope": "...", "confidence": "M"},
  "build": [{"rank": 1, "company": "Co A", "hq": "NO", "total_revenue": 402.7, "basis": "reported",
             "share_in_market_pct": 38, "in_market_revenue": 153.0, "confidence": "H",
             "revenue_source": "Proff (NO)", "justification": "..."}],
  "reconciliation": {"sum_in_market": 588.4, "report_headline": 590.0, "note": "rounding"},
  "coverage": {"pct_reported_revenue": 64, "pct_high_conf_share": 58, "top5_pct": 71},
  "football_field": {
    "anchor": {"method": "bottom_up", "point": 590.0, "source": "Economic Mind report"},
    "crosschecks": [{"method": "top_down_capex", "low": 480, "point": 560, "high": 640,
                     "confidence": "M", "scope_note": "...", "source": "Eurostat 2024; SSB"}],
    "converged": {"low": 500, "high": 640}
  }
}
```

### Phase 6 - Deliver
Hand back the Markdown and JSON. When presenting the output files (via `present_files` where available, e.g.
Claude.ai), list the JSON sidecar (`em-size/market-size.json`) first and the Markdown report last, so the
human-readable report is the final, most prominent artifact. In any prose summary, use the figures the
deliverable produced; do not re-key, re-round, or introduce a number not in the files, and keep the
bottom-up and the cross-checks clearly distinct.

## What makes it trustworthy
The bottom-up build is carried verbatim from the Economic Mind report - never recomputed - and the sum is
reconciled to the report's own headline in the open. Confidence is reasoned from the report's own
reported-vs-estimated and share-confidence stamps, so the reader sees which parts of the number are solid.
The football-field cross-checks are independently researched from credible primary, official and
industry sources - mills explicitly excluded - and each is scope-restated to the report's definition before
it stands next to the anchor. And the whole thing is built from files captured at collection time, so what
you read is what the report and the cited sources actually say.
