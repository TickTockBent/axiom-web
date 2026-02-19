# AXIOM & AXO

**Agent eXecution, Information & Orchestration Markup** and **Agent eXecution Optimization** — Open standards for the agentic web.

---

## The Problem

Every website will face two questions in the next 18 months:

1. **Will your site still work when there is no screen and no human?**
2. **Does your site allow autonomous agents to transact while still blocking the traffic you don't want?**

AI agents are browsing the web on behalf of users right now. When they visit your site, they don't see your design, your layout, or your brand. They see your HTML. If your HTML is an empty shell, the agent moves on. Your competitor gets the sale.

Meanwhile, `robots.txt` — a binary instrument from 1994 — forces site owners to choose between blocking all AI traffic (including agents sending you customers) or allowing all of it (including crawlers harvesting your content for training data).

## The Solution

This repository contains two complementary specifications:

### AXO Framework

**Agent eXecution Optimization** — A scoring and measurement framework that quantifies how effectively AI agents can discover, navigate, understand, and operate a website.

AXO measures agent-readiness across six dimensions:

| Dimension | Weight | What It Measures |
|---|---|---|
| Content Survivability | 25% | Does content exist without JavaScript? |
| Structural Legibility | 20% | Can agents identify page structure from the DOM? |
| Interactive Manifest Clarity | 20% | Can agents find and invoke actions via the accessibility tree? |
| Data Extractability | 15% | Is business data structured for machine consumption? |
| Navigation Traversability | 10% | Can agents explore the site via static links? |
| Agent Response Fitness | 10% | Does the content tell a coherent story as plain text? |

AXO answers: **"How agent-ready is this site?"**

### AXIOM Specification

**Agent eXecution, Information & Orchestration Markup** — An implementation standard for building websites that AI agents can effectively discover, navigate, understand, and operate.

AXIOM provides:

- **Traffic Governance** — A three-tier model that separates crawlers from agents, giving site owners granular control over automated access.
- **An agent manifest** — `axiom.json`, a machine-readable declaration of a site's agent-facing capabilities and access policy.
- **A markup vocabulary** — `data-axiom-*` attributes that communicate intent, structure, and interaction contracts to agents beyond what HTML and ARIA alone provide.
- **Readiness levels** — A tiered adoption path from quick wins (deploy one JSON file) to full agent optimization.

AXIOM answers: **"How do I build an agent-ready site?"**

### How They Work Together

AXO identifies the gap. AXIOM closes it.

| | AXO | AXIOM |
|---|---|---|
| **Purpose** | Diagnostic scoring | Implementation standard |
| **Audience** | Site owners, auditors | Developers, framework authors, CMS builders |
| **Output** | Scores, findings, gap analysis | Manifest spec, markup patterns, readiness levels |

## Repository Structure

```
spec/
  AXO-Framework-v3.md          # The AXO scoring and measurement framework
  AXIOM-Specification-v3.md     # The AXIOM implementation specification
audits/
  examples/                     # Example AXO audit reports
  template.md                   # Blank template for conducting AXO audits
```

## Quick Start: 5 Minutes to AXIOM Level 1

Create `axiom.json` at your domain root:

```json
{
  "axiom_version": "1.0",
  "site": {
    "name": "Your Company",
    "description": "What your business does in one sentence.",
    "primary_language": "en"
  },
  "capabilities": {
    "actions": [
      {
        "id": "contact",
        "description": "Send a message to our team.",
        "entry_point": "/contact",
        "method": "form_submit"
      }
    ]
  },
  "navigation": {
    "sections": [
      { "name": "Home", "path": "/" },
      { "name": "Products", "path": "/products" },
      { "name": "About", "path": "/about" },
      { "name": "Contact", "path": "/contact" }
    ],
    "sitemap": "/sitemap.xml"
  },
  "agent_policy": {
    "tier2_allowed": true,
    "tier3_allowed": true,
    "max_requests_per_minute": 30
  }
}
```

Review your `robots.txt`. Verify `/sitemap.xml` exists. You are now Level 1 AXIOM-ready.

See the full [AXIOM Specification](spec/AXIOM-Specification-v3.md) for Levels 2 and 3.

## Related Tooling

### Charlotte

**[Charlotte](https://github.com/TickTockBent/charlotte)** is an MCP (Model Context Protocol) server that transforms web pages into structured, agent-readable formats. It maintains a persistent headless Chromium browser instance and acts as a translation layer between visual web content and text-based AI reasoning, decomposing pages into semantic representations including accessibility trees, layout information, interactive elements, and content summaries.

Charlotte was used as the primary browser automation tool during AXO audit development and is well-suited for conducting AXO audits programmatically. It enables AI agents to navigate, observe, interact with, and analyze web pages — the same capabilities needed to evaluate a site's agent-readiness across all six AXO dimensions.

## Contributing

AXIOM and AXO are living specifications. Feedback, audit reports, and implementation experience are welcome.

- Open an issue to report problems or suggest improvements to either specification.
- Submit example audit reports to the `audits/` directory.
- If you're a framework author implementing AXIOM patterns, we'd like to hear about it.

## Author

Wesley Shoffner, [Clocktower & Associates](https://www.clocktowerassoc.com)

## License

This project is licensed under the [MIT License](LICENSE).
