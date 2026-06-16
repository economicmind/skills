---
name: company-at-a-glance
description: >
  Produce a rigorous, fully-sourced data snapshot of a single company - a detailed multi-year financial
  table (revenue, gross profit, EBITDA, EBIT, net income per year), computed margins and growth with every
  derived figure footnoted to its formula and inputs, plus web-sourced profile facts (founded, CEO,
  ownership, employees, listing). Use whenever the user wants a company "at a glance", a snapshot / fact
  sheet / profile, "pull the financials for X", "what do we know about X", "give me the numbers on X", or
  names a single company to size up - even if they don't say "at a glance". Financials come ONLY from the
  Economic Mind financials connector and are flagged reported-vs-not-reported; profile facts are
  web-searched from credible sources and individually cited. The deliverable is data (Markdown + JSON);
  presentation is left to the client.
---

# Company at a glance

Produce a one-page, **fully-sourced** snapshot of a single company. The deliverable is **data** - a
Markdown report plus a JSON sidecar - not a styled document; how it is laid out is the client's job. The
entire value is **trust**: every number is either reported (with its provider named) or shown as `n/a`,
every derived figure carries a footnote with its formula and the exact inputs, and every profile fact is
attributed to the source it came from. A reader should be able to audit any figure in the snapshot back to
its origin without taking anything on faith.

## Data sources (no exceptions)

There are **two source channels, and they are never mixed**. Keeping them separate is what lets a reader
trust the numbers: the financials stay pristine and machine-traceable; the qualitative colour is clearly
labelled as such.

1. **Financials & figures - ONLY the Economic Mind connector.** Every revenue, profit, margin and growth
   figure must trace to a `get_company_financials` call. Do **not** pull a financial number from the web,
   from memory, from a market report, or from a "sanity check" - even if the connector is slow, returns
   nothing, or seems wrong. If Economic Mind does not report a figure, it is `n/a`. There is no fallback.
2. **Profile facts - web search, credible sources only.** Founded, CEO, ownership, employees, listing
   status and a short "about" are qualitative context. Web-search them (see Phase 3), cite each one
   individually, and **never let a web-sourced fact enter a calculation.** If you cannot source a fact
   from a credible source, it is `n/a` - do not guess.

## Work to files first, then read them back

The connector response - not your memory of it - is the single source of truth, because context can be
compressed mid-task and a half-remembered number is exactly how errors creep in. So: **immediately after
each tool call returns, write its raw output to a file**, before making the next call. Then, before you
assemble the snapshot, **read every file back** and build the report from the files, not from recall. Use
a working directory you create once at the start, e.g. `em-glance/`.

## Calculations are explicit, gated, and verified

You compute the derived figures yourself - there is no bundled script - so the discipline has to come from
method, not from a black box:

- **Gate on reported inputs.** Compute a figure **only when every input it needs is reported** for the
  relevant year(s). If EBIT is not reported for FY2024, EBIT margin for FY2024 is `n/a` - never infer it.
- **Show the work.** Every derived figure gets a footnote with the formula and the actual numbers
  substituted in, e.g. `EBITDA margin (FY2024) = EBITDA ÷ Revenue = 19.4 ÷ 121.6 = 15.9%`. A figure a
  reader cannot recompute from its footnote does not belong in the snapshot.
- **Verify before you use.** For any multi-step figure (e.g. a CAGR), write the steps out and check the
  intermediate result before relying on it. When in doubt, recompute from the raw file rather than reusing
  a number you calculated earlier.

### Calculation methods (use these exact definitions)
All margins are expressed as % of revenue for the same fiscal year:
- `Gross margin (FYn) = Gross profit ÷ Revenue`
- `EBITDA margin (FYn) = EBITDA ÷ Revenue`
- `EBIT margin (FYn) = EBIT ÷ Revenue`
- `Net margin (FYn) = Net income ÷ Revenue`

Growth (only across years where revenue is reported at both endpoints):
- `Revenue YoY (FYn) = (Revenue_FYn − Revenue_FYn-1) ÷ Revenue_FYn-1`
- `Revenue CAGR, full span = (Revenue_latest ÷ Revenue_earliest)^(1 ÷ (Y−1)) − 1`, where `Y` = number of
  reported revenue years in the span.
- `Revenue CAGR, L3Y = (Revenue_FYn ÷ Revenue_FYn-3)^(1 ÷ 3) − 1` (needs four consecutive reported years).

State the exact years used in every growth footnote, because spans differ when filings are incomplete.

## Source attribution: name the underlying provider

Economic Mind aggregates filings from underlying providers, and each line item carries its own `source`
(e.g. "Proff (NO)", "S&P Capital IQ"). Name the **underlying provider** and say it was gathered through
Economic Mind - e.g. *"Company filings via Proff (NO), gathered through Economic Mind"* - rather than
crediting "Economic Mind" as if it were the primary source. This mirrors how the figure is actually
sourced and lets the reader trace it.

## Workflow

### Phase 1 - Resolve the company & set up
1. Create the working directory once: `mkdir -p em-glance`.
2. `search_companies(["<name>"])`. Pick the entity matching the user's intent, using `headquarters` and
   `summary` to disambiguate. If two are plausible, say which you chose and why; if genuinely ambiguous,
   ask before continuing.
3. Write `em-glance/company-info.txt` with: `companyId`, `companyName`, `headquarters`, the EM `summary`,
   and the currency you will use.

### Phase 2 - Pull the financials (one currency, five years)
1. Request the **five most recent years** including the latest:
   `get_company_financials([id], years=[2021,2022,2023,2024,2025], currency=<one currency>)`. Use one
   currency so the years are comparable. (Adjust the year list over time; the latest EM year is the most
   recent one returned.) If fewer years are reported, that is fine - the snapshot shows exactly what is
   filed.
2. **Immediately write the raw connector JSON** to `em-glance/<id>-financials.json` - before anything else.
3. From that JSON, write a tidy `em-glance/financials.csv`, one row per (year, line_item), keeping the
   per-item provider so attribution survives. Leave the value blank where a line item is not reported -
   do not fill it. Include the source on every row:
   ```
   year,line_item,value,source
   2024,revenue,121.6,Proff (NO)
   2024,gross_profit,,not reported
   2024,ebitda,19.4,Proff (NO)
   2024,ebit,18.6,Proff (NO)
   2024,net_income,14.1,Proff (NO)
   2023,revenue,105.9,Proff (NO)
   ...
   ```

### Phase 3 - Web-search the profile facts (cited)
Collect what you can **with a source for each**: founded, CEO, ownership, employees, listing status, and a
2-3 sentence "about". Start with the EM `summary` (attribute it as such), then **actively web-search for
the facts EM does not carry** - especially CEO and employee count.

Rules for these searches (this is the only place the web is used, and only for qualitative context):
- Use the `WebSearch` / `web_fetch` tools only. Prefer **credible primary sources**: the company's own
  website / About / leadership page, official registries (e.g. Brønnøysund/Proff for Norway, Companies
  House for the UK, SEC filings), annual reports, or established business press (Reuters, Bloomberg, FT,
  national broadsheets). Avoid scraped aggregators (random LinkedIn-scraper sites, ZoomInfo-style estimate
  pages, open wikis) unless nothing better exists - and if you must use one, label the fact an estimate.
- **No hallucination.** If a credible source does not give you the fact, write `n/a` and move on. Never
  invent a CEO, a headcount, or a founding year, and never carry one over from a similarly-named company.
  A confident-looking guess is worse than an honest `n/a`. Ranges are fine if sourced as such (`~250`).
- Write `em-glance/profile-facts.txt`, one fact per block, capturing the **source URL**:
  ```
  FACT: Founded
  VALUE: 1938
  SOURCE: Sperre.com – company history
  URL: https://www.sperre.com/about
  ---
  FACT: CEO
  VALUE: n/a
  SOURCE:
  URL:
  ```

### Phase 4 - Verification & calculations (do not skip)
Work from the files, not from memory.
1. Read every file back: `cat em-glance/company-info.txt`, `cat em-glance/financials.csv`,
   `cat em-glance/profile-facts.txt`.
2. Compute the derived figures from the CSV using the definitions above, **only where the inputs are
   reported**. For each, capture the formula and the exact inputs in `em-glance/calculations.csv`:
   ```
   metric,value,formula,components
   ebitda_margin_FY2024,15.9%,EBITDA÷Revenue,"EBITDA=19.4,Revenue=121.6"
   rev_cagr_L3Y,+0.8%,(Rev_FY2024÷Rev_FY2021)^(1/3)−1,"Rev_FY2024=121.6,Rev_FY2021=118.8"
   ```
3. Cross-check: re-substitute each formula and confirm it yields the stated value; confirm every figure's
   inputs are actually present (not blank) in `financials.csv`; confirm each growth figure names real,
   reported endpoint years. If a check fails, fix the figure or mark it `n/a` - do not ship a number you
   could not verify.

### Phase 5 - Assemble the deliverable
Build the Markdown snapshot and a JSON sidecar **from the files**. Do not re-key a number from memory; pull
every value from `financials.csv` / `calculations.csv` and every fact from `profile-facts.txt`.

Markdown structure (this is data, so keep it plain - the client styles it):
```
# <Company> - company at a glance
*Snapshot generated <date>. Financials reported via <provider(s)>, gathered through Economic Mind, and
computed deterministically; profile facts web-sourced and individually cited. AI-generated - confirm
against the underlying filings before relying on it for a decision.*

## About
<2–3 sentences>  (Source: <where it came from>)

## Key facts
- **Founded:** 1938  (Sperre.com – company history)
- **CEO:** n/a
- **Employees:** ~250  (Proff.no register, 2024)
- **Ownership:** … (…)
- **Listing:** … (…)

## Reported financials by year (<currency> m)
| Line item    | FY2021 | FY2022 | FY2023 | FY2024 | FY2025 |
|--------------|-------:|-------:|-------:|-------:|-------:|
| Revenue      |  118.8 |  155.3 |  105.9 |  121.6 |   n/a  |
| Gross profit |    n/a |    n/a |    n/a |    n/a |   n/a  |
| EBITDA       |   28.5 |   32.6 |   19.1 |   19.4 |   n/a  |
| EBIT         |    …   |    …   |    …   |   18.6 |   n/a  |
| Net income   |    …   |    …   |    …   |   14.1 |   n/a  |
*Blank/n/a = not reported by Economic Mind. Source per line: <provider>.*

## Key figures
- **Latest revenue (FY2024):** <cur> 121.6 m
- **EBITDA margin (FY2024):** 15.9% [^1]
- **EBIT margin (FY2024):** 15.3% [^2]
- **Revenue CAGR (L3Y, FY2021–24):** +0.8% [^3]
*(any figure whose inputs are unreported is shown as n/a, not omitted silently)*

## Calculation footnotes
[^1]: EBITDA margin (FY2024) = EBITDA ÷ Revenue = 19.4 ÷ 121.6 = 15.9%
[^2]: EBIT margin (FY2024) = EBIT ÷ Revenue = 18.6 ÷ 121.6 = 15.3%
[^3]: Revenue CAGR L3Y = (121.6 ÷ 118.8)^(1/3) − 1 = +0.8%

## Sources & method
- **Financials** - <line items> reported by <provider(s)> (e.g. Proff (NO)), gathered through Economic
  Mind; margins and CAGR computed from those reported figures (formulas in footnotes).
- **<each profile fact>** - <source>, <URL>.
- **Not reported by Economic Mind:** <list the gaps, e.g. gross profit all years; FY2025 not yet filed>.
```

JSON sidecar (`em-glance/<id>-glance.json`) - a machine-readable mirror so the client can render without
re-parsing prose:
```json
{
  "company": "Sperre AS",
  "currency": "NOK",
  "about": {"text": "...", "source": "..."},
  "facts": [{"k": "Founded", "v": "1938", "src": "Sperre.com – company history", "url": "..."}],
  "financials": [{"year": 2024, "revenue": 121.6, "ebitda": 19.4, "ebit": 18.6, "net_income": 14.1,
                  "gross_profit": null, "source": "Proff (NO)"}],
  "figures": [{"metric": "ebitda_margin", "year": 2024, "value": 15.9,
               "formula": "EBITDA÷Revenue", "inputs": {"EBITDA": 19.4, "Revenue": 121.6}}],
  "not_reported": ["gross_profit (all years)", "FY2025 (not yet filed)"]
}
```

### Phase 6 - Deliver
Hand back the Markdown and the JSON. When presenting the output files (via `present_files` where available,
e.g. Claude.ai), list the JSON sidecar (`em-glance/<id>-glance.json`) first and the Markdown snapshot last,
so the human-readable snapshot is the final, most prominent artifact. In any prose summary, reference the
figures the snapshot produced - do not re-key or re-round numbers, and do not introduce a figure that is not
in the files.

## What makes it trustworthy
Every financial number is reported (provider named) or `n/a`; nothing is sourced from the web or memory.
Every derived figure is gated on reported inputs and footnoted with its formula and the exact numbers, so
it is independently recomputable. Every profile fact is web-searched from a credible source and cited with
its URL, and never touches a calculation. And the whole snapshot is built from files written at collection
time, not from potentially-compressed recall - so what you read is what was filed.
