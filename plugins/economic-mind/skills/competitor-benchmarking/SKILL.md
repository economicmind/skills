---
name: competitor-benchmarking
description: >
  Benchmark a set of companies against each other on hard financials - a detailed per-year data table for
  each company (revenue, gross profit, EBITDA, EBIT, net income), plus a cross-company summary of size,
  growth and profitability, with every derived figure footnoted to its formula and inputs and every line
  attributed to its underlying provider. Use whenever the user wants to "compare / benchmark companies",
  "how does X stack up against its peers", "build a comp set / comp table", "competitive benchmarking",
  "peer financials for the X market", or names several companies to put side by side. Either accept an
  explicit list of companies or derive the peer set from a target company's Economic Mind market report.
  Financials come ONLY from the Economic Mind connector, are normalized to one currency, and are flagged
  reported-vs-not-reported. The deliverable is data (Markdown + JSON); presentation is left to the client.
---

# Competitor benchmarking

Put a set of companies side by side on their **reported financials**. The deliverable is **data** - a
Markdown report plus a JSON sidecar - not a styled document; layout is the client's job. The entire value
is **trust and comparability**: every number is either reported (with its provider named) or shown as
`n/a`, every derived figure carries a footnote with its formula and exact inputs, every company is pulled
in the **same currency** so sizes are comparable, and the peer set is chosen transparently. A reader
should be able to audit any cell in the benchmark back to its origin.

This is the multi-company sibling of the company-at-a-glance skill and follows the same disciplines. The
hard part of benchmarking is not arithmetic - it is making sure you are comparing like with like, so the
comparability rules below carry as much weight as the calculations.

## Data sources (no exceptions)

**Financials come ONLY from the Economic Mind connector.** Every revenue, profit, margin and growth figure
must trace to a `get_company_financials` call. Do **not** pull a financial number from the web, from
memory, from a market report's prose, or from a "sanity check" - even if the connector is slow, returns
nothing, or seems wrong. If Economic Mind does not report a figure for a company, that cell is `n/a`. There
is no fallback. (The market report is used **only** to choose the peer set - never as a source of
financial figures.)

## Comparability: the three things that make a benchmark honest

A side-by-side table invites the reader to draw conclusions from differences between companies, so those
differences must be real and not artifacts. Three rules:

1. **One currency, for everyone.** Pull every company in the **same** currency (caller-specified, else a
   sensible default - typically the currency of the market report or the one shared by most of the set).
   Economic Mind does the conversion. State the currency and that figures are FX-converted by EM. Without
   this, a "bigger" company might just be a stronger-currency company.
2. **Margins are currency-neutral; growth is not.** A margin is a ratio of two same-year, same-currency
   figures, so it is unaffected by the conversion. A **growth rate or CAGR computed on FX-converted figures
   blends business growth with currency moves.** So compute revenue growth / CAGR from each company's
   **native filing-currency** revenue series, and footnote it as such. If you only have the common-currency
   series, you may still report CAGR but must flag it as *"FX-inclusive"* so the reader knows. (Margins and
   size are fine in the common currency.)
3. **Same year basis.** Companies file on different calendars and report different spans. Always show the
   fiscal year per figure, compute each company's growth across **its own** reported endpoints (name them),
   and never compare a 4-year CAGR for one company against a 2-year CAGR for another without labelling it.

## Work to files first, then read them back

Context can be compressed mid-task, and a half-remembered number across a dozen companies is exactly how a
benchmark goes wrong. So **immediately after each tool call returns, write its raw output to a file**,
before the next call. Then, before assembling the benchmark, **read every file back** and build the report
from the files, not from recall. Create one working directory at the start, e.g. `em-bench/`.

## Calculations are explicit, gated, and verified

You compute the derived figures yourself - there is no bundled script - so the rigor comes from method:

- **Gate on reported inputs.** Compute a figure **only when every input it needs is reported** for the
  relevant year(s) for that company. If EBIT is missing for a company-year, that EBIT margin is `n/a` -
  never infer it, and never borrow a peer's value.
- **Show the work.** Every derived figure gets a footnote with the formula and the actual numbers, e.g.
  `EBITDA margin (Co A, FY2024) = EBITDA ÷ Revenue = 19.4 ÷ 121.6 = 15.9%`. A figure a reader cannot
  recompute from its footnote does not belong in the benchmark.
- **Verify before you use.** For any multi-step figure (a CAGR), write the steps out and check the
  intermediate before relying on it. When in doubt, recompute from the raw file rather than reuse an
  earlier result.

### Calculation methods (use these exact definitions)
Margins, per company per fiscal year (currency-neutral):
- `Gross margin = Gross profit ÷ Revenue`
- `EBITDA margin = EBITDA ÷ Revenue`
- `EBIT margin = EBIT ÷ Revenue`
- `Net margin = Net income ÷ Revenue`

Growth, per company (from the **native-currency** revenue series; name the endpoint years):
- `Revenue YoY (FYn) = (Revenue_FYn − Revenue_FYn-1) ÷ Revenue_FYn-1`
- `Revenue CAGR, full span = (Revenue_latest ÷ Revenue_earliest)^(1 ÷ (Y−1)) − 1`, `Y` = reported revenue
  years in the span.
- `Revenue CAGR, L3Y = (Revenue_FYn ÷ Revenue_FYn-3)^(1 ÷ 3) − 1` (needs four consecutive reported years).

This skill does **not** compute a peer-set median or rank unless the caller asks; keep the summary to
per-company figures so the reader draws their own comparison. (Sorting the table by latest revenue is fine
and helpful - it is not a derived statistic.)

## Source attribution: name the underlying provider, per company

Economic Mind aggregates filings from underlying providers, and the provider can differ **by company and
even by line item** (e.g. "Proff (NO)", "S&P Capital IQ"). This matters more in a benchmark than in a
single profile, because comparability depends on it. Name the **underlying provider per company** and say
it was gathered through Economic Mind - e.g. *"Company filings via Proff (NO), gathered through Economic
Mind"*. If two companies' figures come from different providers, that is worth noting.

## Workflow

### Phase 1 - Resolve the benchmark set
Create the working directory once: `mkdir -p em-bench`. Then build the set one of two ways:

- **Explicit list given:** for each named company, `search_companies(["<name>"])` and pick the right entity
  (disambiguate on `headquarters` / `summary`). Record `companyId` and `companyName`.
- **Target company given (no list):** find the relevant market via `search_markets("<market or target>")`,
  open it with `get_market(...)`, and take its listed players as the candidate set. Resolve each named
  player to an entity via `search_companies`. The market report is used **only** to pick the set.

Either way: pick the common reporting currency now (caller's choice, else the set's dominant currency).
**Confirm the resolved set and currency with the user before pulling everything** - "I'll benchmark these
N companies in <CUR>: …; good to proceed?" - because deriving a wrong or oversized set wastes a lot of
calls. Keep the set manageable (roughly ≤15 unless asked); if a derived set is larger, take the top players
and say so. Write `em-bench/set.txt` (one line per company: id, name, resolved-from).

### Phase 2 - Pull financials (every company, same currency, five years)
For **each** company, request the five most recent years including the latest in the **common currency**:
`get_company_financials([id], years=[2021,2022,2023,2024,2025], currency=<CUR>)`.
**Immediately write the raw JSON** to `em-bench/<id>-financials.json` before the next call.

For growth that is FX-clean (see comparability rule 2), also capture each company's **native-currency**
revenue series - either a second `get_company_financials` call in the company's filing currency, or note
that you will flag CAGR as FX-inclusive if you do not. Write `em-bench/<id>-native.json` when you pull it.

Then consolidate into `em-bench/financials.csv`, one row per (company, year, line_item), keeping the
per-row provider so attribution survives, value blank where not reported:
```
company,year,line_item,value,currency,source
Co A,2024,revenue,121.6,EUR,Proff (NO)
Co A,2024,gross_profit,,EUR,not reported
Co A,2024,ebitda,19.4,EUR,Proff (NO)
Co B,2024,revenue,402.7,EUR,S&P Capital IQ
...
```

### Phase 3 - Verification & calculations (do not skip)
Work from the files, not memory.
1. Read every file back (`cat em-bench/set.txt`, `cat em-bench/financials.csv`, and the native series).
2. For each company, compute margins (common currency) and growth (native currency) using the definitions
   above, **only where inputs are reported**. Capture each in `em-bench/calculations.csv`:
   ```
   company,metric,value,formula,components,currency_basis
   Co A,ebitda_margin_FY2024,15.9%,EBITDA÷Revenue,"EBITDA=19.4,Revenue=121.6",EUR(common)
   Co A,rev_cagr_L3Y,+0.8%,(Rev_FY2024÷Rev_FY2021)^(1/3)−1,"Rev_FY2024=118.0,Rev_FY2021=115.1",NOK(native)
   ```
3. Cross-check: re-substitute each formula and confirm the value; confirm every input is actually present
   (not blank) in `financials.csv`; confirm each growth figure names real reported endpoint years and uses
   the native series (or is flagged FX-inclusive). Fix or mark `n/a` anything you cannot verify - never
   ship an unverifiable cell.

### Phase 4 - Assemble the deliverable
Build the Markdown and JSON **from the files**. Do not re-key a number from memory.

Markdown structure (plain data - the client styles it):
```
# Competitor benchmark - <market / set name> (<CUR> m)
*Generated <date>. N companies, all figures normalized to <CUR> (FX-converted by Economic Mind).
Financials reported via the providers named below, gathered through Economic Mind, and computed
deterministically. Growth computed on native-currency revenue. AI-generated - confirm against the
underlying filings before relying on it.*

## Benchmark summary
| Company | Provider | Latest FY | Revenue | EBITDA | Rev CAGR (L3Y) | Gross margin | EBITDA margin | EBIT margin |
|---------|----------|----------:|--------:|-------:|---------------:|-------------:|--------------:|------------:|
| Co B    | S&P CIQ  | FY2024    |  402.7  |  70.1  | +6.2% [^b1]    |   n/a        | 17.4% [^b2]   | 12.1% [^b3] |
| Co A    | Proff(NO)| FY2024    |  121.6  |  19.4  | +0.8% [^a1]    |   n/a        | 15.9% [^a2]   | 15.3% [^a3] |
*Sorted by latest revenue. Blank/n/a = not reported by Economic Mind. Margins in <CUR> (currency-neutral);
CAGR on native currency.*

## Reported financials by company and year
### Co A  (source: Proff (NO), gathered through Economic Mind)
| Line item    | FY2021 | FY2022 | FY2023 | FY2024 | FY2025 |
|--------------|-------:|-------:|-------:|-------:|-------:|
| Revenue      |  118.8 |  155.3 |  105.9 |  121.6 |   n/a  |
| Gross profit |    n/a |    n/a |    n/a |    n/a |   n/a  |
| EBITDA       |   28.5 |   32.6 |   19.1 |   19.4 |   n/a  |
| EBIT         |    …   |    …   |    …   |   18.6 |   n/a  |
| Net income   |    …   |    …   |    …   |   14.1 |   n/a  |
### Co B  (source: S&P Capital IQ, gathered through Economic Mind)
| … |
*Blank/n/a = not reported by Economic Mind.*

## Calculation footnotes
[^a1]: Revenue CAGR L3Y (Co A, native NOK) = (118.0 ÷ 115.1)^(1/3) − 1 = +0.8%
[^a2]: EBITDA margin (Co A, FY2024) = EBITDA ÷ Revenue = 19.4 ÷ 121.6 = 15.9%
[^a3]: EBIT margin (Co A, FY2024) = EBIT ÷ Revenue = 18.6 ÷ 121.6 = 15.3%
[^b1]: …

## Sources & method
- **Peer set** - <explicit list | derived from the Economic Mind "<market>" report>; N companies.
- **Currency** - all figures in <CUR>, FX-converted by Economic Mind; growth computed on native currency.
- **Financials per company** - Co A via Proff (NO); Co B via S&P Capital IQ; … all gathered through
  Economic Mind; margins/CAGR computed from those reported figures (formulas in footnotes).
- **Not reported by Economic Mind:** Co A gross profit (all years), FY2025 (not yet filed); Co B …
```

JSON sidecar (`em-bench/benchmark.json`) - machine-readable mirror:
```json
{
  "set_name": "Nordic fire protection",
  "currency": "EUR",
  "peer_set_basis": "derived from Economic Mind 'Nordic fire protection' market report",
  "companies": [
    {"company": "Co A", "provider": "Proff (NO)",
     "financials": [{"year": 2024, "revenue": 121.6, "ebitda": 19.4, "ebit": 18.6,
                     "net_income": 14.1, "gross_profit": null}],
     "figures": [{"metric": "ebitda_margin", "year": 2024, "value": 15.9,
                  "formula": "EBITDA÷Revenue", "inputs": {"EBITDA": 19.4, "Revenue": 121.6},
                  "currency_basis": "EUR (common)"},
                 {"metric": "rev_cagr_l3y", "value": 0.8, "currency_basis": "NOK (native)"}],
     "not_reported": ["gross_profit (all years)", "FY2025"]}
  ]
}
```

### Phase 5 - Deliver
Hand back the Markdown and JSON. When presenting the output files (via `present_files` where available, e.g.
Claude.ai), list the JSON sidecar (`em-bench/benchmark.json`) first and the Markdown report last, so the
human-readable report is the final, most prominent artifact. In any prose summary, reference the figures the
benchmark produced - do not re-key or re-round numbers, and do not introduce a figure that is not in the
files. If you want a
single-company deep profile (CEO, ownership, employees, web-sourced facts), that is the company-at-a-glance
skill's job - this skill stays on comparable financials.

## What makes it trustworthy
Every financial number is reported (provider named, per company) or `n/a`; nothing is sourced from the web
or memory. Every company is pulled in the same currency so size is comparable, margins are currency-neutral,
and growth is computed on native currency so it is not distorted by FX. Every derived figure is gated on
reported inputs and footnoted with its formula and exact numbers. The peer set is chosen transparently and
confirmed before pulling. And the whole benchmark is built from files written at collection time, not from
compressed recall - so what you read is what was filed.
