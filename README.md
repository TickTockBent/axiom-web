# ASM, ACP & LEAN

**Three specs for the agentic web. One file to deliver them.**

---

## The Spec Family

| Spec | Full Name | Scope | Deliverable |
|------|-----------|-------|-------------|
| **ASM** | Agent Site Manifest | Site-level | `site` block in `agents.json` |
| **[ACP](https://github.com/Clocktower-and-Associates/ACP)** | Agent Context Protocol | Service-level | `services` block in `agents.json` |
| **LEAN** | Layered Efficiency for Agentic Navigation | Design philosophy | Guides authoring of both |

**One-liner:** Add `/.well-known/agents.json` to your site. That's it. Agents can now understand your property and use your services.

---

## The Problem

Every website will face two questions in the next 18 months:

1. **Will your site still work when there is no screen and no human?**
2. **Does your site allow autonomous agents to transact while still blocking the traffic you don't want?**

AI agents are browsing the web on behalf of users right now. When they visit your site, they don't see your design, your layout, or your brand. They see your HTML. If your HTML is an empty shell, the agent moves on. Your competitor gets the sale.

Meanwhile, `robots.txt` — a binary instrument from 1994 — forces site owners to choose between blocking all AI traffic (including agents sending you customers) or allowing all of it (including crawlers harvesting your content for training data).

## ASM: Agent Site Manifest

ASM tells agents how to navigate and interact with a web property. It has two parts:

### ASM Scoring Framework

A measurement and diagnostic framework that quantifies how effectively AI agents can discover, navigate, understand, and operate a website. It scores agent-readiness across six dimensions:

| Dimension | Weight | What It Measures |
|---|---|---|
| Content Survivability | 25% | Does content exist without JavaScript? |
| Structural Legibility | 20% | Can agents identify page structure from the DOM? |
| Interactive Manifest Clarity | 20% | Can agents find and invoke actions via the accessibility tree? |
| Data Extractability | 15% | Is business data structured for machine consumption? |
| Navigation Traversability | 10% | Can agents explore the site via static links? |
| Agent Response Fitness | 10% | Does the content tell a coherent story as plain text? |

The Scoring Framework answers: **"How agent-ready is this site?"**

### ASM Build Spec

An implementation standard providing:

- **Traffic Governance** — A three-tier model that separates crawlers from agents, giving site owners granular control over automated access.
- **A site manifest** — The `site` block in `agents.json`, a machine-readable declaration of a site's agent-facing capabilities and access policy.
- **A markup vocabulary** — `data-asm-*` attributes that communicate intent, structure, and interaction contracts to agents beyond what HTML and ARIA alone provide.
- **Readiness levels** — A tiered adoption path from quick wins (deploy one JSON file) to full agent optimization.

The Build Spec answers: **"How do I build an agent-ready site?"**

## ACP: Agent Context Protocol

ACP defines how services advertise their capabilities to agents. Instead of requiring agents to run a local MCP server, the service hosts a structured manifest describing available tools, auth requirements, and call conventions. Agents fetch this at runtime and call directly over HTTPS.

- No local server process required
- No SDK dependency
- No context injection at startup
- The service bears the cost of describing itself, not the agent
- Agents discover, fetch, and call directly over HTTPS

ACP content lives in the `services` block of `agents.json`.

See the full [ACP specification](https://github.com/Clocktower-and-Associates/ACP).

## Consolidated Delivery: `/.well-known/agents.json`

Following the `robots.txt` convention: one file, one location, everything an agent needs.

```json
{
  "version": "1.0",
  "site": {
    // ASM content: navigation structure, content hints,
    // interaction patterns, page-level metadata
  },
  "services": {
    // ACP content: tools, auth, endpoints, constraints
  }
}
```

- A website with no API populates only `site`
- A headless API populates only `services`
- A full product populates both
- One fetch, one parse, complete agent context

## Repository Structure

```
spec/
  ASM-Build-Spec-v1.0.md          # Implementation standard
  ASM-Scoring-Framework-v1.0.md   # Measurement and diagnostic framework
audits/
  examples/                        # Example ASM audit reports
  template.md                      # Blank template for conducting ASM audits
```

## Quick Start: 5 Minutes to Level 1

Create `/.well-known/agents.json`:

```json
{
  "version": "1.0",
  "site": {
    "name": "Your Company",
    "description": "What your business does in one sentence.",
    "primary_language": "en",
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
}
```

Review your `robots.txt`. Verify `/sitemap.xml` exists. You are now Level 1 ASM-ready.

See the full [ASM Build Spec](spec/ASM-Build-Spec-v1.0.md) for Levels 2 and 3.

## Related Tooling

### Charlotte

**[Charlotte](https://github.com/TickTockBent/charlotte)** is an MCP (Model Context Protocol) server that transforms web pages into structured, agent-readable formats. It maintains a persistent headless Chromium browser instance and acts as a translation layer between visual web content and text-based AI reasoning, decomposing pages into semantic representations including accessibility trees, layout information, interactive elements, and content summaries.

Charlotte was used as the primary browser automation tool during ASM audit development and is well-suited for conducting ASM audits programmatically. It enables AI agents to navigate, observe, interact with, and analyze web pages — the same capabilities needed to evaluate a site's agent-readiness across all six ASM dimensions.

## Contributing

ASM and ACP are living specifications. Feedback, audit reports, and implementation experience are welcome.

- Open an issue to report problems or suggest improvements.
- Submit example audit reports to the `audits/` directory.
- If you're a framework author implementing ASM patterns, we'd like to hear about it.

## Author

Wesley Shoffner, [Clocktower & Associates](https://www.clocktowerassoc.com)

## License

This project is licensed under the [MIT License](LICENSE).
