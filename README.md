# Skills by Economic Mind

The skills are built for **knowledge workers** - analysts, founders, investors, consultants and
operators - who need **market insights** they can trust. The skills are tightly coupled to the Economic Mind connector. If you do not already have access to it, you can reach out to us [here](https://www.economicmind.com/contact).

## Installation

### Claude (web & desktop app)

1. Open the **Customize** menu in the left sidebar click the **+** next to **Personal plugins**.
2. Hover **Create plugin**, click **Add marketplace**, click **Add from a repository**.
3. Enter the repository `economicmind/skills`.
4. Find the **`Economic Mind`** plugin in the marketplace and click the **+** in the top right corner.
5. Once installed, the plugin should appear under personal plugins on the left sidebar.
6. Under the plugin name, click **Connectors → Install → Connect**. This will open a new window where you can authenticate to the Economic Mind connector.

Once installed, type `/getting-started` in any chat or Cowork to verify everything works as expected and start using the skills.

### Claude Code

```
/plugin marketplace add economicmind/skills
/plugin install economic-mind@skills
```

On first use you'll be prompted to authenticate to the Economic Mind connector (OAuth, via `/mcp`).

### Knowledge work plugin marketplace

We're in the process of applying for a listing on Anthropic's
[**knowledge-work-plugins**](https://github.com/anthropics/knowledge-work-plugins) marketplace. Until that
listing lands, install directly from this repository using the instructions above.

## Enabling it for your organization

On **Team** and **Enterprise** plans, an owner or admin can distribute the Economic Mind plugin to
everyone in the organization through an org-managed marketplace:

1. In **Organization settings**, confirm **Cowork** and **Skills** are enabled.
2. Go to **Organization settings → Plugins** and add the marketplace. Org marketplaces synced from
   GitHub must point at a **private/internal** repository, so fork `economicmind/skills` into your
   org and connect that fork (`owner/repo` format) - or manually upload the plugin.
3. Select the **`Economic Mind`** plugin and set an installation preference:
   - **Installed by default** - auto-installed; members can remove it.
   - **Available for install** - listed in the catalog for self-service.
   - **Required** - auto-installed with no removal option.
   - **Not available** - hidden from members.
4. Enterprise admins can override these per group (e.g. auto-install for Research, available for
   everyone else). Changes take effect on each member's next session.

See [Manage plugins for your organization](https://support.claude.com/en/articles/13837433-manage-plugins-for-your-organization)
for the full admin guide.

## Skills overview

The Economic Mind plugin ships these skills. The three analysis skills each produce **data**
(Markdown + JSON) that's fully sourced and ready to drop into your own work:

- **`company-at-a-glance`** - a fully-sourced snapshot of a single company: multi-year financials
  (revenue, gross profit, EBITDA, EBIT, net income), computed margins and growth - every derived
  figure footnoted to its formula and inputs - plus web-sourced profile facts (founded, CEO,
  ownership, employees, listing).
- **`competitor-benchmarking`** - benchmark a set of companies on hard financials: per-year tables
  per company plus a cross-company summary of size, growth and profitability, normalized to one
  currency, every derived figure footnoted to its formula, and every line attributed to its
  underlying provider.
- **`bottom-up-market-size`** - size a market bottom-up from an Economic Mind market report: the
  transparent per-company build carried verbatim from the report (with its confidence ratings and
  justifications), stress-tested against 2–3 credible, independently-sourced cross-checks.
- **`getting-started`** - an orientation to the plugin and connector: what it is, what data it draws
  on, which skills and tools are available, and a quick connector check. Start here if you're new.

The skills draw their data from the **Economic Mind connector**, bundled with the plugin as a remote
MCP server.
