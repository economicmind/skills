---
name: getting-started
description: >
  Orient a user to the Economic Mind plugin and connector - what it is, what data it draws on, which
  skills and connector tools are available, how to trigger them, and what to expect as the product
  grows. Use whenever the user is new to Economic Mind or asks what it is / what it can do / how to
  use it / how to get started - "what is Economic Mind", "getting started", "help", "what can the
  connector do", "what tools or skills are available", "how do I trigger X", "introduce the plugin",
  or names Economic Mind without a concrete analysis task yet. As part of the orientation, confirm the
  user is connected to the Economic Mind connector and offer one quick live example. Do NOT use this
  when the user already has a concrete request (a company, a peer set, a market to size) - hand off to
  the matching data skill instead.
---

# Getting started with Economic Mind

Economic Mind is a library of **bottom-up market perspectives** and **hard company financials**, served
to Claude through a connector. This plugin adds **skills** that turn that data into rigorous,
fully-sourced analysis - every figure traces back to its source, and the deliverable is data
(Markdown + JSON) that the client is free to style. Your job when this skill runs is to orient the
user: explain what's here, show how to trigger it, confirm the connector is connected, and point them
to the right skill for their goal. Keep it brief and concrete - a tour, not a manual.

## What you get

There are two layers, and it helps to keep them distinct:

1. **The connector** - a remote MCP server (`https://mcp.economicmind.com/mcp`) that exposes Economic
   Mind's data as tools: curated market reports, and multi-year company financials. Claude calls these
   tools automatically when a request needs data; the user does not call them by hand.
2. **The skills** - analysis workflows built on top of the connector. Each one is a disciplined
   pipeline that pulls connector data, works to files, and assembles a fully-sourced deliverable.

## The skills

- **`company-at-a-glance`** - a fully-sourced snapshot of one company: multi-year financials, computed
  margins and growth (each figure footnoted to its formula), plus web-sourced profile facts.
  Try: *"give me Sperre AS at a glance"*, *"pull the financials for <company>"*,
  *"what do we know about <company>"*.
- **`competitor-benchmarking`** - a comp set across several companies: per-year financial tables plus a
  cross-company summary of size, growth and profitability, normalized to one currency.
  Try: *"benchmark <A>, <B> and <C>"*, *"build a comp table for the Nordic fire-protection market"*,
  *"how does <company> stack up against its peers"*.
- **`bottom-up-market-size`** - size a market from an Economic Mind market report (the transparent
  per-company build) and stress-test the headline against credible cross-checks.
  Try: *"size the Nordic 3PL market bottom-up"*, *"how big is <market> and how sure are we"*,
  *"show the market-size buildup for <market>"*.

## The connector tools

These are the data tools the skills (and Claude) draw on. Coverage is broad - market reports span many
topics and geographies, and company financials run across multiple years, with the deepest depth in the
Nordics.

- **`search_markets`** - find market reports by topic and geography; returns each market's headline
  size, scope and content catalog. *"find market reports on autonomous trucking in the Nordics"*.
- **`get_market`** - open a specific market report by its slug and read a section (demand, supply,
  growth, …) in full. *"open that market and show me the demand section"*.
- **`search_companies`** - resolve a company name to Economic Mind's record (id, summary, HQ).
  *"find the company Sperre AS"*.
- **`get_company_financials`** - pull a company's yearly P&L (revenue, EBITDA, EBIT, net income, …) in
  a chosen currency, with each line attributed to its provider. *"pull <company>'s revenue and EBITDA
  for the last three years in NOK"*.

## How to trigger

| Way | How | Example |
|-----|-----|---------|
| Natural language | Just describe what you want - works everywhere | *"size the Nordic 3PL market bottom-up"* |
| Skill by name | Invoke a skill directly | `/economic-mind:company-at-a-glance` |
| Connector tools | Called automatically by Claude when a request needs data | - (you don't call these yourself) |

Natural language is the primary path and the most reliable across surfaces. Invoking a skill by name
is handy when you know exactly which one you want; the exact slash syntax depends on the surface
(Claude Code vs. Claude.ai / Desktop).

## What to expect going forward

Economic Mind is growing. Expect coverage to keep widening - more markets and geographies, and deeper
company and report data - and new connector capabilities to arrive over time. The skills will track the
connector as it grows, so the analysis you can ask for will broaden alongside the data.

## Confirm the connector, then hand off

After orienting the user, confirm the connector is actually connected before sending them off:

1. **Run one lightweight check.** Make a single cheap call - e.g. `search_markets` with a sample query
   the user might care about - to confirm connectivity. Keep it to one call; do not probe exhaustively.
2. **On success**, briefly show a real result as a live example ("here's a market I found -"), then
   suggest a concrete next step: pick one of the three skills with a real prompt.
3. **On an auth or permission error**, tell the user to connect the **Economic Mind** connector and
   retry - in Claude Code via `/mcp`, or in Claude.ai / Desktop via the connector settings. Don't
   retry in a loop; once is enough to surface the prompt.
