# ASM Build Spec

## Implementation Standard for the Agentic Web

**Author:** Wesley Shoffner, Clocktower & Associates
**Version:** 1.0
**Date:** March 2026

---

## 1. The Two Questions

Every website will face two questions in the next 18 months:

1. **Will your site still work when there is no screen and no human?**
2. **Does your site allow autonomous agents to transact while still blocking the traffic you don't want?**

ASM (Agent Site Manifest) answers both.

ASM is an implementation standard for building websites that AI agents can effectively discover, navigate, understand, and operate. Where the ASM Scoring Framework defines **what to measure** and **how to score** agent-readiness, ASM defines **how to build for it**.

ASM provides:

1. **Traffic Governance** — A three-tier model that separates crawlers from agents, giving site owners granular control over automated access.
2. **An agent manifest** — `agents.json`, a machine-readable declaration of a site's agent-facing capabilities and access policy.
3. **A markup vocabulary** — Data attributes that communicate intent, structure, and interaction contracts to agents beyond what HTML and ARIA alone provide.
4. **Readiness levels** — A tiered adoption path from quick wins to full agent optimization.

ASM is additive. It does not replace HTML semantics, ARIA, schema.org, or any existing web standard. It extends them where gaps exist.

---

## 2. Traffic Governance — The Three-Tier Access Model

### 2.1 The robots.txt Crisis

`robots.txt` is a binary instrument from 1994 being used for a spectrum of use cases it was never designed for.

A site owner who wants to block AI training crawlers currently has no way to do so without also blocking the personal agent a customer sends to make a purchase. The `GPTBot` user-agent string is used by both OpenAI's training crawler and OpenAI's Operator agent. `ClaudeBot` identifies both Anthropic's indexing crawler and Anthropic's user-facing assistant. Blocking the user-agent blocks both.

Companies are making this tradeoff right now. Some block all AI access to protect their content from training data extraction. In doing so, they also block every agent-mediated customer interaction — every purchase, every quote request, every booking that a personal agent would have completed on behalf of a user.

**Without ASM, the choice is: block everything or allow everything.** There is no middle ground in `robots.txt`.

### 2.2 The Three Tiers

ASM defines three tiers of automated web traffic, each with different access governance.

#### Tier 1: Crawlers (Indexing, Training, RAG Retrieval)

Automated systems that traverse and index content at scale — search engine crawlers, AI training pipelines, RAG retrieval systems.

**Access governed by: `robots.txt`**

This is what `robots.txt` was designed for. Tier 1 traffic MUST respect `robots.txt` directives. No change to existing conventions. ASM takes no position that overrides `robots.txt` for crawling traffic.

#### Tier 2: Discovery & Comparison Agents

Agents performing research or comparison tasks on behalf of a user — "find me the best price across ten sites," "compare features of these three products," "what restaurants near me have outdoor seating?"

These agents act on user intent but behave like crawlers: visiting many pages, extracting data, and moving on. They occupy a gray zone between indexing and user interaction.

**Access governed by: `agents.json` agent policy**

If `agents.json` is present, Tier 2 agents SHOULD follow its agent policy directives (rate limits, access rules) and MAY disregard `robots.txt` for the purpose of fulfilling user-initiated requests. If `agents.json` is absent, Tier 2 agents SHOULD fall back to respecting `robots.txt`.

If `agent_policy.tier2_allowed` is explicitly set to `false`, Tier 2 agents MUST NOT access the site.

#### Tier 3: User-Directed Task Agents

Agents executing a specific task on behalf of a user who has explicitly chosen to interact with the site — "buy this item," "fill out this contact form," "book an appointment for Tuesday."

The user has directed the agent to a specific site for a specific purpose. Blocking the agent blocks the user.

**Access governed by: `agents.json` agent policy for rate limits and politeness. NOT bound by `robots.txt`.**

Tier 3 agents are functionally equivalent to the user operating a browser. They SHOULD respect agent policy rate limits and politeness directives. They SHOULD NOT be blocked by `robots.txt`, because the intent is user-directed site interaction, not automated crawling.

If `agent_policy.tier3_allowed` is explicitly set to `false`, Tier 3 agents MUST respect this and inform the user that the site does not permit agent-mediated interaction.

### 2.3 The Strategic Value

The three-tier model resolves the binary problem:

| Tier | Traffic Type | Default Access | Override |
|------|-------------|---------------|----------|
| 1 | Crawlers (training, indexing) | `robots.txt` | — |
| 2 | Discovery agents (comparison, research) | Allowed (respect agent policy) | `tier2_allowed: false` |
| 3 | User-directed agents (purchase, booking) | Allowed (respect agent policy) | `tier3_allowed: false` |

This gives site owners the tool they're missing: **"Crawlers stay out. Agents are welcome. Here are the rules."**

`agents.json` is a treaty with AI agents. It's a negotiation file that tells ChatGPT, Claude, Perplexity, and every other AI system: your training crawlers are banned from scraping our content, but your customer-facing agents are welcome to shop, book, and transact — under these terms.

**Without ASM:** You must block all AI traffic — including the agents sending you customers — or allow all of it, including the crawlers harvesting your content for training data.

**With ASM:** You separate business traffic from parasitic traffic. Agents that bring customers are welcomed. Crawlers that extract training data are blocked. The distinction is explicit, machine-readable, and enforceable.

This moves ASM from a developer tool to a strategic asset for legal, security, and business teams.

### 2.4 Conflict Resolution

If `robots.txt` and `agents.json` specify conflicting rate limits, ASM-aware agents SHOULD follow the **more restrictive** directive. If `robots.txt` specifies `Crawl-delay: 10` and `agents.json` specifies `crawl_delay_seconds: 1`, the agent SHOULD use the 10-second delay unless it can identify itself as Tier 2 or Tier 3 traffic and the site has explicitly opted into ASM's tiered model by publishing `agents.json`.

The presence of `agents.json` signals that the site owner has made a deliberate choice about agent access. In this case, the agent policy takes precedence over `robots.txt` for Tier 2 and Tier 3 traffic.

### 2.5 Future Considerations

The current model governs access at the site level. Path-level granularity — "Tier 2 allowed for product pages but not for account pages" — is a recognized need that will be addressed in a future specification version. The site-level model is sufficient for initial adoption and covers the majority of current use cases.

---

## 3. The Agent Manifest — `agents.json`

### 3.1 Overview

`agents.json` is a machine-readable file hosted at `/.well-known/agents.json` — following the same convention as other `.well-known` resources — that declares a site's agent-facing capabilities, navigation structure, and access policy. A single file serves both ASM (site manifest) and ACP (service protocol) content.

It answers the question every agent has upon arriving at a site: *"What can I do here, and how do I do it?"*

**This is the minimum viable adoption of ASM.** A site can reach Level 1 readiness (see Section 7) by deploying `agents.json` and reviewing `robots.txt`. No HTML changes. No new attributes. One file.

### 3.2 Location & Discovery

The manifest MUST be accessible at:

```
https://example.com/.well-known/agents.json
```

#### Primary Discovery: Dual-Layer `<head>` Approach

Current LLM-based agents process the full HTML of a page, including comments. A dual-layer approach in the document `<head>` ensures discovery by both today's agents and tomorrow's:

```html
<!-- AI agents: check /.well-known/agents.json for site navigation and operation instructions -->
<link rel="agents-manifest" type="application/json" href="/.well-known/agents.json">
```

**Layer 1: The HTML Comment (Works Today)**

A natural language directive like `AI agents: check /.well-known/agents.json for site navigation and operation instructions` is immediately understood by any language model parsing the page. It requires no prior training, no convention recognition, no ecosystem buy-in. This works with every LLM-based agent that can read HTML, right now.

**Layer 2: The `<link>` Tag (Long-Term Standard)**

```html
<link rel="agents-manifest" type="application/json" href="/.well-known/agents.json">
```

This follows established web conventions (`<link rel="manifest">` for PWAs, `<link rel="sitemap">` for sitemaps, `<link rel="icon">` for favicons). It's the correct long-term mechanism — machine-parseable, semantic, and consistent with how the web handles discoverability. Once agent providers begin checking for `rel="agents-manifest"` natively, this becomes the primary discovery path.

**Transition Path:**

| Phase | Discovery Method | Status |
|-------|-----------------|--------|
| **Now** | HTML comment | Primary — works with all LLM-based agents today |
| **Adoption** | Both comment + `<link>` | Belt and suspenders during transition |
| **Mature** | `<link>` only | Comment becomes redundant and can be removed |

#### Supplementary Discovery Mechanisms

These can supplement the `<head>` approach but are less reliable for current LLM-based agents:

- **`robots.txt` directive** — `Agents: /.well-known/agents.json` (mirrors the `Sitemap:` convention). Useful if agents check `robots.txt` before fetching pages.
- **HTTP response header** — `Link: </.well-known/agents.json>; rel="agents-manifest"`. Doesn't require touching HTML, but agents must inspect headers.
- **`.well-known` path** — The file is already at `/.well-known/agents.json`, following IETF convention.

The `<head>` dual-layer approach is recommended as the primary mechanism because it places the directive inside content that agents are already reading.

### 3.3 Manifest Structure

The `agents.json` file serves both ASM (site manifest) and ACP (service protocol) content in a single file. A website with no API populates only `site`. A headless API populates only `services`. A full product populates both. One fetch, one parse, complete agent context.

The full manifest below shows every available field in the `site` block. **Most are optional.** A Level 1 manifest requires only `version`, `site.name`, `site.description`, `site.capabilities`, `site.navigation`, and `site.agent_policy` — roughly 25 lines of JSON. See Section 8.1 for the minimal quick-start template.

```jsonc
{
  // Required. agents.json format version.
  "version": "1.0",

  // ASM content — site-level manifest (see ASM spec).
  "site": {
    // Required. Human-readable site identity.
    "name": "Acme Corp",
    "description": "Enterprise widget manufacturing and distribution.",
    "primary_language": "en",
    "contact": "https://www.acme.com/contact",
    "sameAs": [
      "https://www.linkedin.com/company/acme-corp",
      "https://github.com/acme-corp"
    ],

    // Required. What agents can do on this site.
    "capabilities": {
      "actions": [
        {
          "id": "search_products",
          "description": "Search the product catalog by keyword, category, or specification.",
          "entry_point": "/products",
          "method": "form_submit",
          "input_schema": {
            "query": { "type": "string", "required": true, "description": "Search terms" },
            "category": { "type": "string", "required": false, "enum": ["widgets", "gizmos", "components"] }
          }
        },
        {
          "id": "get_quote",
          "description": "Request a price quote for a product configuration.",
          "entry_point": "/quote",
          "method": "form_submit",
          "input_schema": {
            "product_id": { "type": "string", "required": true },
            "quantity": { "type": "integer", "required": true, "min": 1 }
          },
          "authentication_required": true
        },
        {
          "id": "contact_sales",
          "description": "Submit a message to the sales team.",
          "entry_point": "/contact",
          "method": "form_submit",
          "input_schema": {
            "name": { "type": "string", "required": true },
            "email": { "type": "email", "required": true },
            "message": { "type": "string", "required": true }
          }
        }
      ],

      "data_types": [
        {
          "type": "product",
          "description": "Product listings with specifications, pricing, and availability.",
          "schema": "https://schema.org/Product",
          "locations": ["/products", "/products/*"]
        },
        {
          "type": "faq",
          "description": "Frequently asked questions about products and services.",
          "schema": "https://schema.org/FAQPage",
          "locations": ["/faq", "/support/faq"]
        }
      ]
    },

    // Required. Site structure for agent navigation.
    "navigation": {
      "sections": [
        {
          "name": "Products",
          "path": "/products",
          "description": "Full product catalog with specifications and pricing.",
          "children": [
            { "name": "Widgets", "path": "/products/widgets" },
            { "name": "Gizmos", "path": "/products/gizmos" },
            { "name": "Components", "path": "/products/components" }
          ]
        },
        {
          "name": "Pricing",
          "path": "/pricing",
          "description": "Pricing plans and volume discount schedules."
        },
        {
          "name": "About",
          "path": "/about",
          "description": "Company history, team, and mission."
        },
        {
          "name": "Contact",
          "path": "/contact",
          "description": "Sales inquiries, support requests, and office locations."
        }
      ],
      "sitemap": "/sitemap.xml"
    },

    // Optional. Authentication and access requirements.
    "access": {
      "public_content": true,
      "authentication": {
        "required_for": ["get_quote", "order_placement"],
        "methods": ["session_cookie", "api_key"],
        "login_url": "/login"
      }
    },

    // Optional. Technical context for agent interaction.
    "technical": {
      "rendering": "ssr",
      "spa_framework": null,
      "content_survivability": "full",
      "api_available": false,
      "api_documentation": null
    },

    // Optional. Agent access policy (see Section 2: Traffic Governance).
    "agent_policy": {
      "crawl_delay_seconds": 1,
      "max_requests_per_minute": 30,
      "tier2_allowed": true,
      "tier3_allowed": true
    }
  },

  // ACP content — service-level protocol (see ACP spec). Optional.
  "services": {
    // See ACP specification for structure.
  }
}
```

### 3.4 Manifest Fields Reference

All fields below are within the `site` block of `agents.json`. The `services` block is defined by the ACP specification.

#### `site` identity fields (Required)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Human-readable site name |
| `description` | string | Yes | One-sentence description of what this site/business does |
| `primary_language` | string | Yes | ISO 639-1 language code |
| `contact` | string | No | URL to contact page or email |
| `sameAs` | array | No | URIs identifying this entity on other platforms (LinkedIn, GitHub, Crunchbase, etc.) for entity reconciliation |

#### `site.capabilities.actions` (Required, array)

Each action declares something an agent can do on the site.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier for the action |
| `description` | string | Yes | Plain-language description of what the action does |
| `entry_point` | string | Yes | URL path where the action can be initiated |
| `method` | enum | Yes | `form_submit`, `link_follow`, or `api_call` |
| `input_schema` | object | No | Schema of inputs the action accepts |
| `authentication_required` | boolean | No | Whether the action requires authentication (default: false) |
| `output_description` | string | No | What the agent should expect after completing the action |

#### `site.capabilities.data_types` (Optional, array)

Declares what structured data agents can extract.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | Yes | Descriptive name of the data type |
| `description` | string | Yes | What this data represents |
| `schema` | string | No | schema.org or other vocabulary URL |
| `locations` | array | Yes | URL patterns where this data type appears |

#### `site.navigation.sections` (Required, array)

Hierarchical site structure for agent orientation.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Section name |
| `path` | string | Yes | URL path |
| `description` | string | No | What agents will find in this section |
| `children` | array | No | Nested subsections |

#### `site.access` (Optional)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `public_content` | boolean | Yes | Whether any content is publicly accessible |
| `authentication.required_for` | array | No | Action IDs that require auth |
| `authentication.methods` | array | No | Supported auth mechanisms |
| `authentication.login_url` | string | No | URL of login page |

#### `site.technical` (Optional)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `rendering` | enum | No | `ssr`, `csr`, `ssg`, or `hybrid` |
| `spa_framework` | string | No | Frontend framework if applicable |
| `content_survivability` | enum | No | `full` (all content in initial HTML), `partial` (some content requires JS), or `none` (empty shell without JS) |
| `api_available` | boolean | No | Whether a public API exists |
| `api_documentation` | string | No | URL to API docs |

#### `site.agent_policy` (Optional)

Access rules and politeness directives for agent traffic. See Section 2 for the three-tier access model.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `crawl_delay_seconds` | number | No | Minimum seconds between requests for agent traffic |
| `max_requests_per_minute` | number | No | Rate limit ceiling for agent traffic |
| `tier2_allowed` | boolean | No | Whether discovery/comparison agents are permitted (default: true) |
| `tier3_allowed` | boolean | No | Whether user-directed task agents are permitted (default: true) |

---

## 4. Design Principles

### 4.1 Progressive Enhancement

ASM layers on top of existing best practices. A site that uses semantic HTML, implements schema.org, and maintains a clean `robots.txt` is already partially agent-ready. ASM adds agent-specific capabilities without requiring sites to abandon or replace anything they already do.

### 4.2 Graceful Degradation

Every ASM addition degrades gracefully for user agents that don't understand it. Data attributes are ignored by browsers. The manifest is a separate file that only agents request. No ASM pattern breaks the human experience.

### 4.3 Discoverability Over Convention

Agents should not need to guess. ASM prefers explicit declaration over implicit convention. If a site has an agent-relevant capability, ASM provides a way to declare it unambiguously.

### 4.4 Minimal Surface Area

ASM targets the smallest vocabulary that addresses real gaps. It does not duplicate what semantic HTML, ARIA, and schema.org already solve. Every ASM pattern justifies its existence by addressing a capability gap identified by the ASM Scoring Framework.

---

## 5. Markup Vocabulary — `data-asm-*`

ASM introduces a set of `data-asm-*` attributes for cases where semantic HTML and ARIA don't provide sufficient signal for agent interaction. These attributes are supplementary — they never replace proper semantic markup.

**These attributes are optional.** A site can achieve Level 1 and Level 2 readiness (see Section 7) without using any `data-asm-*` attributes. The markup vocabulary is a Level 3 optimization for organizations that want maximum agent capability.

### 5.1 Page-Level Attributes

Applied to the `<body>` or top-level container. Tells the agent what kind of page this is and what it can accomplish here.

```html
<body data-asm-page-type="product-listing"
      data-asm-page-purpose="Browse and compare widget products by category and specification.">
```

| Attribute | Description |
|-----------|-------------|
| `data-asm-page-type` | Classifies the page: `homepage`, `product-listing`, `product-detail`, `article`, `contact`, `checkout`, `search-results`, `faq`, `landing`, `documentation`, `account` |
| `data-asm-page-purpose` | Plain-language sentence describing what an agent can accomplish on this page |

### 5.2 Section-Level Attributes

Applied to content sections to communicate intent and priority.

```html
<section data-asm-role="primary-content"
         data-asm-summary="Product specifications and pricing for the X100 Widget.">
  ...
</section>

<aside data-asm-role="supplementary"
       data-asm-summary="Related products and cross-sell recommendations.">
  ...
</aside>
```

| Attribute | Description |
|-----------|-------------|
| `data-asm-role` | Agent-facing role: `primary-content`, `supplementary`, `navigation`, `metadata`, `promotional`, `legal`, `interactive` |
| `data-asm-summary` | Plain-language summary of the section's content |
| `data-asm-priority` | Relative importance: `critical`, `high`, `medium`, `low` |

### 5.3 Action Attributes

Applied to interactive elements to communicate intent beyond what ARIA provides. The `data-asm-consequences` attribute is the most novel addition — nothing else in the existing standards ecosystem tells an agent what happens when it clicks a button.

```html
<button data-asm-action="add-to-cart"
        data-asm-target="product:X100"
        data-asm-consequences="Item added to cart. Cart count increments. Cart drawer opens."
        aria-label="Add X100 Widget to cart">
  Add to Cart
</button>

<a href="/pricing"
   data-asm-action="navigate"
   data-asm-intent="view-pricing">
  See Pricing
</a>
```

| Attribute | Description |
|-----------|-------------|
| `data-asm-action` | Declares the action type: `add-to-cart`, `submit-form`, `navigate`, `toggle`, `filter`, `sort`, `search`, `authenticate`, `download`, `contact`, `share` |
| `data-asm-target` | What the action operates on. Format: `type:identifier` (e.g., `product:X100`, `form:contact`) |
| `data-asm-consequences` | Plain-language description of what happens when this action is invoked. Critical for agents deciding whether to act. |
| `data-asm-intent` | The purpose of following a link, when `href` alone doesn't convey intent |
| `data-asm-confirmation` | Whether this action requires confirmation: `true` or `false` |

### 5.4 Data Attributes

Applied to data-bearing elements to communicate structure and semantics.

```html
<div data-asm-data-type="pricing-table"
     data-asm-data-key="product:X100"
     data-asm-data-freshness="2026-02-15">
  <dl>
    <dt>Base Price</dt>
    <dd data-asm-data-field="base_price"
        data-asm-data-value="299.99"
        data-asm-data-unit="USD">$299.99</dd>
    <dt>Volume Discount (100+)</dt>
    <dd data-asm-data-field="volume_price_100"
        data-asm-data-value="249.99"
        data-asm-data-unit="USD">$249.99</dd>
  </dl>
</div>
```

| Attribute | Description |
|-----------|-------------|
| `data-asm-data-type` | Type of structured data: `pricing-table`, `spec-sheet`, `comparison`, `schedule`, `contact-info`, `inventory`, `review-summary` |
| `data-asm-data-key` | Identifier for the entity this data describes (format: `type:id`) |
| `data-asm-data-field` | Field name for machine extraction |
| `data-asm-data-value` | Machine-parseable value (separate from display formatting) |
| `data-asm-data-unit` | Unit of measurement: `USD`, `EUR`, `kg`, `cm`, `percent`, etc. |
| `data-asm-data-freshness` | ISO 8601 date indicating when this data was last verified/updated |

### 5.5 State Attributes

Applied to elements whose state affects agent decision-making but isn't covered by ARIA state attributes.

```html
<div data-asm-state="in-stock"
     data-asm-state-label="In Stock — ships within 2 business days">
  <span class="stock-badge green">●</span> In Stock
</div>
```

| Attribute | Description |
|-----------|-------------|
| `data-asm-state` | Machine-readable state value: `in-stock`, `out-of-stock`, `limited`, `pre-order`, `active`, `inactive`, `archived`, `sale`, `new` |
| `data-asm-state-label` | Plain-language state description. Eliminates reliance on color or icon interpretation. |

### 5.6 Visibility Attributes

Applied to elements that exist in the DOM but should be treated differently by agents.

```html
<div class="cookie-banner" data-asm-visibility="dismiss"
     data-asm-dismiss-action="click:#accept-cookies">
  We use cookies...
  <button id="accept-cookies">Accept</button>
</div>

<!-- When the dismiss target IS this element, use click:self -->
<button class="promo-close" data-asm-visibility="dismiss"
        data-asm-dismiss-action="click:self">
  ✕ Close
</button>

<div class="promo-overlay" data-asm-visibility="ignore">
  Sign up for 10% off!
</div>
```

| Attribute | Description |
|-----------|-------------|
| `data-asm-visibility` | Agent visibility directive: `ignore` (skip entirely), `dismiss` (dismiss and proceed), `defer` (low priority, process only if relevant) |
| `data-asm-dismiss-action` | How to dismiss: `click:self` (click this element), `click:<selector>` (click target element), `escape`, `scroll`, `wait:<seconds>`. Selectors SHOULD use element IDs for stability across DOM changes (e.g., `click:#accept-cookies` not `click:.modal-footer > button:first-child`). |

---

## 6. Remediation Patterns

Each pattern maps to an ASM dimension and addresses a specific failure mode. Patterns are ordered from highest to lowest impact within each dimension.

### 6.1 Content Survivability Patterns

**CS-1: Server-Side Render Critical Content**

If the site uses a client-rendered SPA, ensure critical business content (product information, pricing, contact details, primary CTAs) is server-rendered or statically generated. Use SSR frameworks (Next.js, Nuxt, SvelteKit, Astro) or pre-rendering.

**CS-2: Meaningful `<noscript>` Fallbacks**

For client-rendered sections, provide `<noscript>` content that communicates the purpose of the section and directs agents to alternative data sources.

```html
<noscript>
  <p>This section requires JavaScript to display interactive product configurator.
  Product specifications are available at <a href="/products/x100/specs">/products/x100/specs</a>.</p>
</noscript>
```

**CS-3: Declare Rendering Strategy**

Include `rendering` in `agents.json` technical section. Agents can adjust their strategy based on whether the site is SSR, CSR, SSG, or hybrid.

### 6.2 Structural Legibility Patterns

**SL-1: Landmark Coverage**

Every page MUST have: `<header>`, `<nav>`, `<main>`, `<footer>`. Use `<aside>` for sidebars, `<article>` for self-contained content, `<section>` for thematic grouping. Label landmarks with `aria-label` when multiples of the same type exist.

```html
<nav aria-label="Primary navigation">...</nav>
<nav aria-label="Footer navigation">...</nav>
```

**SL-2: Sequential Heading Hierarchy**

Every page MUST have exactly one `<h1>`. Subsequent headings follow sequential order without skipping levels.

```
✓ h1 → h2 → h3 → h3 → h2 → h3
✗ h1 → h3 → h5 (skipped levels)
✗ h1 → h1 (duplicate h1)
```

**SL-3: Semantic Ratio Target**

Target a semantic-to-generic element ratio above 15%. Replace layout `<div>` elements with `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>` wherever the element has a structural purpose.

**SL-4: Language Declaration**

Always set `lang` attribute on `<html>`. For multilingual content, set `lang` on content-level elements that differ from the page language.

### 6.3 Interactive Manifest Clarity Patterns

**IMC-1: Native Interactive Elements**

Use `<button>` for actions, `<a>` for navigation, `<input>`/`<select>`/`<textarea>` for data entry. If custom elements are necessary, add `role`, `tabindex="0"`, and keyboard event handlers.

**IMC-2: Accessible Name Coverage**

Every interactive element MUST have an accessible name. Priority order: visible text content > `<label>` association > `aria-label` > `aria-labelledby`. Accessible names should describe the action, not the appearance ("Submit contact form" not "Blue button").

**IMC-3: State Attributes on Stateful Controls**

All controls with state MUST communicate that state via ARIA:

| Control | Required Attributes |
|---------|-------------------|
| Dropdown/accordion | `aria-expanded="true|false"` |
| Toggle | `aria-pressed="true|false"` |
| Tabs | `aria-selected="true|false"` |
| Menu triggers | `aria-haspopup="true|menu|dialog"` |
| Disabled controls | `aria-disabled="true"` |
| Checkboxes (custom) | `aria-checked="true|false|mixed"` |

**IMC-4: ASM Action Declarations** *(Level 3)*

For primary actions, add `data-asm-action` and `data-asm-consequences` to give agents explicit understanding of what an interaction will do.

**IMC-5: `aria-live` for Dynamic Content**

Regions that update dynamically MUST have `aria-live="polite"` or `aria-live="assertive"` so agents can detect changes without polling.

### 6.4 Data Extractability Patterns

**DE-1: Semantic Data Elements**

Use `<table>` for tabular data (with `<thead>`, `<th>`, `<caption>`), `<dl>` for key-value pairs, `<ol>`/`<ul>` for lists. Never use `<div>` grids as a substitute for data tables.

**DE-2: Schema.org Structured Data**

Implement JSON-LD structured data for all business-critical content. Minimum coverage:

| Content Type | Schema |
|-------------|--------|
| Organization info | `Organization` |
| Products | `Product` (with `Offer`, `AggregateRating`) |
| Articles/blog posts | `Article` or `BlogPosting` |
| FAQs | `FAQPage` with `Question`/`Answer` |
| Events | `Event` |
| Reviews | `Review` |
| Breadcrumbs | `BreadcrumbList` |
| Search | `WebSite` with `SearchAction` |

**DE-3: Machine-Readable Values**

Dates MUST use `<time datetime="...">`. Prices MUST be in text (not images). Use `data-asm-data-value` and `data-asm-data-unit` where the display format differs from the machine-parseable value. *(Level 3)*

**DE-4: Consistent Data Patterns**

Repeated content types (product cards, listing items, search results) MUST use consistent DOM structure. An agent that learns the pattern for one product card should be able to extract data from all product cards.

### 6.5 Navigation Traversability Patterns

**NT-1: XML Sitemap**

Publish a comprehensive XML sitemap at `/sitemap.xml`. Reference it in `robots.txt`. Update it when content changes.

**NT-2: Breadcrumb Navigation**

Implement breadcrumbs on all pages below the homepage. Mark up with `BreadcrumbList` schema.

**NT-3: Static Link Coverage**

All primary navigation paths MUST use standard `<a href="...">` links that resolve without JavaScript. Hamburger/mobile menus must have `<a>` tags on every item even when the menu is visually collapsed.

**NT-4: AI Crawler Access & Agent Policy**

Review `robots.txt` for Tier 1 (crawler) access. Publish `agents.json` with agent policy to provide granular control over Tier 2 (discovery) and Tier 3 (user-directed) agent traffic.

Common AI user-agent strings include `GPTBot` (OpenAI), `ClaudeBot` (Anthropic), `PerplexityBot` (Perplexity), and `Google-Extended` (Google AI). However, these strings currently do not distinguish between tiers — OpenAI's Operator (a Tier 3 user-directed agent) uses the same `GPTBot` header as OpenAI's training crawler (Tier 1). This conflation is part of the problem ASM addresses: `robots.txt` blocks by user-agent, but the same user-agent may represent fundamentally different access patterns. `agents.json` provides tier-based control that `robots.txt` cannot.

**NT-5: Descriptive Link Text**

Links MUST use descriptive text that communicates the destination or action.

**NT-6: Pagination Over Infinite Scroll**

Content lists MUST be navigable via standard paginated links. Infinite scroll or "load more" may be offered as an enhancement but must not be the only access pattern.

### 6.6 Agent Response Fitness Patterns

**ARF-1: Information-First Content Order**

In DOM order, the most important business information should appear early. Site purpose should be identifiable within the first 200 words of the text stream.

**ARF-2: Descriptive Headings**

Headings MUST summarize the content that follows. An agent reading only the headings should be able to produce an accurate page outline.

**ARF-3: Image Alt Text**

Informational images MUST have alt text that describes the information the image conveys, not the image's appearance. Decorative images MUST have `alt=""` and `aria-hidden="true"`.

**ARF-4: ASM Visibility Directives** *(Level 3)*

Mark non-content overlays (cookie banners, promo modals, chat widgets) with `data-asm-visibility="dismiss"` or `data-asm-visibility="ignore"`.

**ARF-5: Eliminate Duplicate Content**

Avoid duplicating heading or CTA text in `aria-label` attributes when the visible text is already sufficient.

---

## 7. Readiness Levels

ASM defines three readiness levels, providing a progressive adoption path. Each level has a clear effort boundary and a clear value proposition.

### Level 1: Manifest & Access (Quick Wins — Days, Not Weeks)

**What you do:** Deploy `/.well-known/agents.json`. Review and fix `robots.txt`. No HTML changes required.

**Why it matters:** Agents arriving at your site immediately have a structured map of your capabilities, navigation, and access policy. You gain control over which automated traffic is welcome and which isn't. Sites that were previously blocking all AI traffic can now welcome agent customers while continuing to block training crawlers.

| Requirement | Effort |
|------------|--------|
| Publish `/.well-known/agents.json` with `site` block containing identity, `capabilities`, and `navigation` | Write one JSON file |
| Configure `agent_policy` in the `site` block (tier2/tier3 access, rate limits) | Add one section to JSON |
| Review `robots.txt` — ensure AI crawlers are not unintentionally blocked | Edit one line |
| Verify XML sitemap exists at `/sitemap.xml` | Check one file |

**Expected ASM score improvement:** +5–15 points on Navigation Traversability. Primary value is agent orientation and traffic governance rather than score improvement.

### Level 2: Structural Readiness (HTML Hygiene — What You Should Already Be Doing)

**What you do:** Fix the HTML fundamentals. Semantic landmarks, heading hierarchy, native interactive elements, schema.org structured data. This is standard web development best practice — if your site doesn't already do these things, it has bigger problems than agent readiness.

**Why it matters:** Agents can now identify your page structure, find your interactive elements, and extract structured data. The site goes from "parseable with heuristics" to "machine-readable by design."

| Requirement | ASM Dimension |
|------------|---------------|
| All Level 1 requirements | — |
| Semantic HTML landmarks on every page (`<header>`, `<nav>`, `<main>`, `<footer>`) | Structural Legibility |
| Sequential heading hierarchy with single `<h1>` | Structural Legibility |
| `lang` attribute on `<html>` | Structural Legibility |
| Server-side render all critical business content (or meaningful `<noscript>` fallbacks) | Content Survivability |
| Native interactive elements for all primary actions | Interactive Manifest Clarity |
| Accessible names on all interactive elements | Interactive Manifest Clarity |
| All primary navigation uses `<a href>` links | Navigation Traversability |
| Descriptive link text on all navigation links | Navigation Traversability |
| Schema.org JSON-LD for Organization, primary content type, and BreadcrumbList | Data Extractability |
| `<time datetime>` on all dates | Data Extractability |
| Semantic tables for tabular data | Data Extractability |
| ARIA state attributes on all stateful controls | Interactive Manifest Clarity |
| Breadcrumb navigation on subpages | Navigation Traversability |
| Cookie/promo overlays dismissable or marked `aria-hidden` | Agent Response Fitness |

**Expected ASM score at Level 2: 70–89 (Grade B)**

### Level 3: Agent-Optimized (Full ASM — Competitive Advantage)

**What you do:** Add `data-asm-*` attributes to communicate action intent, data structure, state, and visibility directives to agents. This is the optimization layer for organizations that want maximum agent capability and are investing in the agentic web as a strategic channel.

**Why it matters:** Agents don't just understand your site — they can operate it with confidence. They know what happens when they click a button, what data they can extract, what state an element is in, and what content to skip. This is the difference between "an agent can probably figure out your site" and "an agent can reliably transact on your site."

| Requirement | ASM Dimension |
|------------|---------------|
| All Level 2 requirements | — |
| `data-asm-page-type` and `data-asm-page-purpose` on all pages | Agent Response Fitness |
| `data-asm-action` and `data-asm-consequences` on primary CTAs | Interactive Manifest Clarity |
| `data-asm-data-*` attributes on business-critical data elements | Data Extractability |
| `data-asm-visibility` on overlays and non-content elements | Agent Response Fitness |
| `data-asm-state` on elements with visual-only state indicators | Agent Response Fitness, Data Extractability |
| Consistent DOM patterns across repeated content types | Data Extractability |
| Link depth ≤ 3 clicks from homepage to any content page | Navigation Traversability |
| Information-first DOM ordering | Agent Response Fitness |
| Comprehensive schema.org coverage (all applicable content types) | Data Extractability |
| Semantic-to-generic element ratio > 15% | Structural Legibility |

**Expected ASM score at Level 3: 90–100 (Grade A)**

---

## 8. Implementation Guide

### 8.1 Quick Start (5 Minutes to Level 1)

**Step 1:** Create `/.well-known/agents.json`. Start with this minimal template:

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

**Step 2:** Review your `robots.txt`. If you're blocking `GPTBot`, `ClaudeBot`, or other AI user-agents, decide whether that's intentional. If you want to block training crawlers but welcome customer agents, `agents.json` now expresses that distinction.

**Step 3:** Verify `/sitemap.xml` exists and is current.

You are now Level 1 ASM-ready.

### 8.2 For Developers

**Audit your templates.** Most sites are generated from a small number of templates. Fix the template, fix every page that uses it.

**Schema.org is the highest-ROI investment.** It improves ASM Data Extractability scores and makes your content more useful to any system that consumes structured data — search engines, AI models, and agents alike.

**Don't remove JavaScript — augment the baseline.** ASM doesn't require abandoning client-side rendering. It requires ensuring critical content is available without it. SSR/SSG/hybrid approaches let you keep your architecture while meeting Content Survivability requirements.

### 8.3 For Framework Authors

**Ship semantic defaults.** Component libraries should render semantic HTML by default (`<button>` not `<div role="button">`).

**Generate `agents.json` from route definitions.** Frameworks with file-based routing (Next.js, Nuxt, SvelteKit, Astro) already have the navigation structure — expose it as an ASM manifest. An `@asm/next` plugin that reads the `app/` directory, generates `agents.json`, adds `data-asm-page-type` from route naming conventions, and infers `data-asm-action` from form elements would get sites to 80% of Level 3 readiness with a single `npm install`.

**Provide ASM helpers.** Consider an `<AsmAction>` wrapper component that adds `data-asm-action` and `data-asm-consequences` attributes.

### 8.4 For CMS Platforms

**Include `agents.json` generation** in site build pipelines. Content types map to `capabilities.data_types`. Navigation structures map to `navigation.sections`.

**Default `robots.txt` should allow AI crawlers.** Many CMS platforms ship with restrictive defaults that block GPTBot, ClaudeBot, etc.

**Schema.org should be automatic.** Generate JSON-LD from content type definitions and field mappings.

---

## 9. Relationship to Existing Standards

| Standard | What It Covers | ASM Relationship |
|---------|---------------|-------------------|
| **HTML5 Semantics** | Document structure and meaning | ASM requires HTML5 semantics as a foundation and extends where gaps exist for agent interaction |
| **ARIA** | Accessible rich internet applications | ASM requires proper ARIA usage and adds `data-asm-*` attributes for agent-specific context beyond accessibility |
| **schema.org** | Structured data vocabulary | ASM requires schema.org for data extractability and adds `data-asm-data-*` for inline data annotation |
| **robots.txt** | Crawler access control | ASM respects robots.txt for Tier 1 (crawler) traffic and provides agent policy for granular Tier 2/3 (agent) access control |
| **sitemap.xml** | Content discovery | ASM requires sitemaps and adds `navigation.sections` in `agents.json` for hierarchical site structure |
| **MCP** | LLM ↔ tool integration | MCP defines how agents connect to tools. ASM defines what agents find when those tools fetch a web page. Complementary. |
| **A2A** | Agent ↔ agent communication | A2A governs how agents talk to each other. ASM governs how agents interact with websites. Different layers. |
| **AGENTS.md** | Developer-facing agent instructions | `agents.json` provides structured, machine-parseable capabilities and governance where AGENTS.md provides prose guidance. Complementary. |
| **llms.txt** | LLM content guidance | `agents.json` provides structured, machine-parseable capabilities and navigation where `llms.txt` provides prose guidance. Complementary. |
| **WCAG 2.1** | Human accessibility | ASM shares the same foundation (semantic HTML, accessibility tree) but measures and optimizes for machine consumers with different needs |

---

## 10. Relationship to ASM Scoring Framework

| | ASM Scoring Framework | ASM Build Spec |
|--|-----|-------|
| **Document** | ASM Scoring Framework | ASM Build Spec |
| **Purpose** | Scoring & measurement | Implementation standard |
| **Audience** | Site owners, auditors | Developers, framework authors, CMS builders |
| **Output** | Scores, findings, gap analysis | Manifest spec, markup patterns, readiness levels |
| **Question** | "How agent-ready is this site?" | "How do I build an agent-ready site?" |

The Scoring Framework identifies the gap. The Build Spec closes it. Together they form the diagnostic-and-treatment framework for the agentic web.

---

## Versioning

ASM is a living specification. Changes are versioned and backward-compatible within major versions.

The `version` field in `agents.json` declares which specification version the manifest conforms to.

---

## License and Attribution

The ASM Build Spec is developed and maintained by Wesley Shoffner at Clocktower & Associates.

The specification is published for industry adoption. Framework authors, CMS platforms, and development teams are encouraged to implement ASM patterns and contribute feedback.

For audit services, implementation consulting, or tooling: [Clocktower & Associates](https://www.clocktowerassoc.com)
