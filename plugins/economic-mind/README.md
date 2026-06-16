# Economic Mind (plugin)

The `economic-mind` plugin for Claude.

It ships a set of **data skills** that turn Economic Mind connector data into rigorous,
fully-sourced financial and market analysis. Every figure traces back to its source, and the skills
produce data whose presentation format is up to the user:

- **`/company-at-a-glance`** - a fully-sourced snapshot of a single company: multi-year financials
  (revenue, gross profit, EBITDA, EBIT, net income), computed margins and growth, plus web-sourced
  profile facts (founded, CEO, ownership, employees, listing).
- **`/competitor-benchmarking`** - benchmark a set of companies on hard financials: per-year tables
  per company plus a cross-company summary of size, growth and profitability, normalized to one
  currency and attributed to each underlying provider.
- **`/bottom-up-market-size`** - size a market bottom-up from an Economic Mind market report (the
  transparent per-company build) and stress-test the headline against 2–3 credible cross-checks.

New to the plugin? Start with **`/getting-started`** for an orientation to the skills and connector,
plus a quick connector check.

The skills draw data from the **Economic Mind connector**, which the plugin bundles as a remote MCP
server (`.mcp.json`) - so installing the plugin wires up both the skills and the data tools.

## Install

See the [root README](../../README.md) for install instructions (Claude Code and Claude Desktop).
