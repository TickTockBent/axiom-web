# ASM Scoring Framework

## Measurement & Diagnostics for the Agentic Web

**Author:** Wesley Shoffner, Clocktower & Associates
**Version:** 1.0
**Date:** March 2026

---

## 1. The Core Question

**Your site has two interfaces. You only built one of them.**

Your HTML is an API you never designed. Every time an agent issues a GET request to your domain, it consumes that API — parsing your DOM for content, structure, actions, and data. Unlike a browser, it doesn't render CSS. Unlike a human, it doesn't interpret visual cues. It reads what you served and acts on what it found.

Frontier AI agents — Operator, Comet, Dia — can afford to render your page in a headless browser. They have vision models, JavaScript runtimes, and million-dollar infrastructure. They're also a rounding error. The actual agentic web is the long tail: thousands of simple agents making raw HTTP requests and parsing whatever comes back. MCP tool chains. Comparison bots. Local assistants. API wrappers hitting fifty sites per query. Every new MCP server, every new API integration, every new personal assistant adds another agent to that long tail — and none of them are spinning up a browser to read your page.

**The web already solved this problem once.** Semantic HTML provides a baseline that works before CSS makes it beautiful and JavaScript makes it interactive. Every capable browser gets the enhanced experience. Every constrained browser still gets the content. ASM applies the same principle to a new audience: your site needs to work for the cheapest, simplest agent that will ever visit it, the same way it needs to work without CSS for the most constrained browser.

The ASM Scoring Framework quantifies how effectively AI agents can discover, navigate, understand, and operate a website. It measures the gap between what humans see and what agents can actually use — across six dimensions, with quantitative scores and letter grades.

---

## 2. The Invisible Economy

### 2.1 The Agent Landscape

The headlines focus on frontier agents — the products with names and launch dates:

- **January 2025:** OpenAI launches Operator, a browser-using agent for task execution.
- **Mid-2025:** Agentic browsers emerge — Perplexity Comet, Browser Company Dia, Opera Neon.
- **July 2025:** OpenAI integrates Operator into ChatGPT as "agent mode," available to all paid users.

These agents can render your page. They have headless browsers, vision models, and the infrastructure to execute JavaScript. They'll handle your site fine.

**They're also not the problem.**

For every frontier agent with a headless browser, there are hundreds of lightweight agents that do a `fetch()` and parse the response. MCP tool chains that hit your page as part of a larger workflow. Comparison services that query fifty sites per request. Local assistants running on consumer hardware. Open-source agents with no budget for browser automation. Vertical-specific bots built by a developer with an API key and a weekend.

This ratio gets more extreme over time, not less. Every new MCP server adds agents to the long tail. Every API wrapper, every local assistant, every task-specific bot. The diversity of agent capabilities is exploding while the floor — raw HTTP GET, parse the DOM — stays the same.

**Your site needs to work for the floor.**

### 2.2 The Inverted Conversion Funnel

For human visitors, conversion is driven by visual design, emotional appeal, social proof, and urgency. None of those things exist for an agent.

For agent-mediated interactions, conversion is driven by:

- **Can the agent see your content?** (Content Survivability)
- **Can the agent understand your page structure?** (Structural Legibility)
- **Can the agent find and invoke your purchase action?** (Interactive Manifest Clarity)
- **Can the agent extract your product data reliably?** (Data Extractability)

A site optimized exclusively for human conversion may be completely non-functional when an agent is making purchasing or recommendation decisions on behalf of a user.

This is not a future problem. It is a present one.

### 2.3 What a Bad Score Costs You

When an agent can't operate your site, it doesn't show an error page. It doesn't bounce. It silently substitutes your competitor.

"I found several options, but I was unable to access the purchasing options for [Your Brand]. I have added [Competitor Brand] to your cart instead."

The user never sees your site. They never see your brand. They see a completed transaction with someone else. You lost a high-intent, ready-to-buy customer and you have no analytics event that tells you it happened.

The question isn't "what percentage of my traffic is agent-mediated?" The question is "how many different kinds of agents are trying to access my site, and how many of them fail silently?" The number of agent *types* is growing exponentially — every new MCP server, every new tool integration, every new personal assistant. Each one that fails on your site is a customer interaction that never happened and was never logged.

---

## 3. The Six Dimensions

ASM measures agent-readiness across six dimensions. Each dimension is scored independently. The dimensions are ordered from most fundamental (can the agent see anything at all?) to most sophisticated (does the content tell a coherent story?).

### 3.1 Content Survivability (Weight: 25%)

**Core question:** What percentage of the site's meaningful content remains visible and functional without JavaScript execution?

**Why this is the primary differentiator:** This is the dimension that separates ASM from every other web audit discipline. Accessibility auditors test with JavaScript enabled — screen readers operate on the rendered page. SEO crawlers execute JavaScript. Performance tools measure rendered output.

But agents frequently don't render JavaScript at all. They issue a GET request and parse the raw HTML response. It's faster, cheaper, and more reliable than spinning up a headless browser. If your site is a client-rendered single-page application, that GET request returns an empty `<div id="app"></div>` and nothing else. Your site is invisible. Not slow. Not broken. *Invisible.*

A fully WCAG-compliant React SPA can score 100% on accessibility and score **zero** on Content Survivability.

**Measurement criteria:**

| Signal | Method |
|--------|--------|
| Text content ratio (no-JS vs. full render) | Compare word count and element count in raw HTML response vs. after full JS execution |
| Critical business info in no-JS state | Check for presence of pricing, contact, product descriptions, CTAs in initial HTML |
| Navigation without JavaScript | Test primary nav links for valid `href` attributes vs. JS-only event handlers |
| `<noscript>` fallbacks | Evaluate presence and quality of `<noscript>` content — empty, redirect, or meaningful? |
| Server-side rendering coverage | Identify SSR/SSG framework markers; compare initial HTML completeness |

**Scoring bands:**

| Score | Condition |
|-------|-----------|
| 90–100 | ≥95% of content present in initial HTML. Fully SSR or static. |
| 70–89 | ≥70% of content present. Critical business info survives. Some JS-enhanced sections. |
| 50–69 | 40–69% of content present. Partial SSR. Key information gaps without JS. |
| 25–49 | 10–39% of content present. Shell HTML with some server-rendered fragments. |
| 0–24 | <10% of content present. Empty shell or loader-only initial HTML. |

### 3.2 Structural Legibility (Weight: 20%)

**Core question:** How easily can an agent identify the core layout — navigation, main content, sidebar, footer — directly from the DOM structure?

**Why it matters:** Agents need to distinguish primary content from chrome, navigation from body, and sidebar from main. Semantic HTML landmarks provide this structure natively. Sites built with generic `<div>` elements and CSS-only layout cues force agents to infer structure heuristically, which is unreliable.

**Measurement criteria:**

| Signal | Method |
|--------|--------|
| Landmark coverage | Check for `<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`, `<article>`, `<section>` |
| Heading hierarchy | Validate sequential H1–H6 structure. Flag skipped levels and missing H1. |
| ARIA landmark roles | Identify `role` attributes on non-semantic containers |
| Semantic-to-generic ratio | Count semantic elements vs. `<div>` and `<span>` containers |
| DOM order vs. visual order | Compare DOM sequence to rendered layout for content reordering via CSS |
| `lang` attribute | Verify presence on `<html>` element |
| Landmark labeling | Check `aria-label` or `aria-labelledby` on landmarks, especially when multiples of the same type exist |

**Scoring bands:**

| Score | Condition |
|-------|-----------|
| 90–100 | Full landmark coverage, clean heading hierarchy, semantic ratio >30%, `lang` present |
| 70–89 | Most landmarks present, minor heading gaps, semantic ratio >15% |
| 50–69 | Partial landmark usage, heading hierarchy breaks, semantic ratio 8–15% |
| 25–49 | Minimal landmarks, significant heading gaps, semantic ratio 3–7% |
| 0–24 | No landmarks, no heading structure, semantic ratio <3% ("div soup") |

### 3.3 Interactive Manifest Clarity (Weight: 20%)

**Core question:** Can an agent find and invoke the site's key actions — CTAs, forms, interactive controls — through the accessibility tree?

**Why it matters:** The accessibility tree is the most reliable programmatic interface for discovering what actions a page offers. If interactive elements are implemented as styled `<div>` elements with JavaScript click handlers rather than native `<button>`, `<a>`, or `<input>` elements, they may be invisible to agents.

**Measurement criteria:**

| Signal | Method |
|--------|--------|
| Native element usage | Percentage of interactive elements using `<button>`, `<a>`, `<input>`, `<select>`, `<textarea>` vs. custom elements |
| Accessible name coverage | Percentage of interactive elements with accessible names (visible labels, `aria-label`, `aria-labelledby`) |
| CTA discoverability | Whether primary CTAs appear in the accessibility tree with meaningful labels |
| Form field associations | Percentage of form inputs with properly linked `<label>` elements |
| State communication | Presence of `aria-expanded`, `aria-selected`, `aria-disabled`, `aria-pressed`, `aria-checked` on stateful elements |
| Keyboard operability | Whether all interactive elements are reachable and operable via keyboard |
| Generic clickable elements | Count of `<div>` or `<span>` elements with click handlers but no `role` or `tabindex` |

**Scoring bands:**

| Score | Condition |
|-------|-----------|
| 90–100 | All interactions use native elements, ≥98% have accessible names, state attributes on all stateful controls |
| 70–89 | Most interactions native, ≥85% named, some state attribute gaps |
| 50–69 | Mixed native/custom, 60–84% named, limited state communication |
| 25–49 | Significant custom elements without ARIA, <60% named, minimal state attributes |
| 0–24 | Primary CTAs are non-semantic, widespread unnamed controls, no state feedback |

### 3.4 Data Extractability (Weight: 15%)

**Core question:** How well is business-critical data structured in semantic HTML that agents can accurately parse and extract?

**Why it matters:** When an agent is comparing products, checking prices, evaluating business information, or summarizing offerings on behalf of a user, it needs to extract structured data reliably. Pricing in an image, product specs in unstructured prose, or tabular data rendered as styled `<div>` grids are all extraction barriers.

**Measurement criteria:**

| Signal | Method |
|--------|--------|
| Semantic data elements | Use of `<table>`, `<dl>`, `<ul>`/`<ol>` for structured data vs. `<div>` grids |
| Structured data markup | Presence and validity of schema.org (JSON-LD, microdata, or RDFa) |
| Table quality | `<thead>`, `<th>`, `<caption>`, `scope` attributes on data tables |
| Machine-readable dates | Use of `<time datetime="...">` vs. unstructured text dates |
| Data consistency | Whether similar content types (e.g., product cards) use consistent DOM patterns |
| Critical data in text | Pricing, availability, contact info as parseable text vs. images, SVGs, or canvas |
| Key-value patterns | Use of `<dl>`/`<dt>`/`<dd>` or equivalent structured patterns for attribute data |

**Scoring bands:**

| Score | Condition |
|-------|-----------|
| 90–100 | Comprehensive schema.org, semantic tables/lists, machine-readable dates, consistent patterns |
| 70–89 | Valid schema.org coverage, mostly semantic data elements, minor gaps |
| 50–69 | Basic schema.org (WebSite only), some semantic elements, inconsistent patterns |
| 25–49 | Minimal or no schema.org, presentation-driven data layout, some text-based data |
| 0–24 | Critical data in images/SVGs, no structured data, no semantic data elements |

### 3.5 Navigation Traversability (Weight: 10%)

**Core question:** Can an agent systematically explore the site via static links without requiring complex JavaScript interactions?

**Why it matters:** Agents explore sites by following links. If navigation requires hover interactions, JavaScript-triggered menus, infinite scroll to reveal content, or authentication walls before any content is visible, the agent's ability to discover and traverse content is severely limited.

**Measurement criteria:**

| Signal | Method |
|--------|--------|
| Sitemap availability | Check for XML sitemap at `/sitemap.xml` and reference in `robots.txt` |
| Static link coverage | Percentage of primary navigation paths that use standard `<a href>` links |
| Breadcrumb navigation | Presence of breadcrumbs with structured data (BreadcrumbList schema) |
| Link depth | Maximum clicks from homepage to reach any content page |
| Pagination patterns | Standard link-based pagination vs. infinite scroll or "load more" buttons |
| `robots.txt` configuration | Whether AI crawlers (GPTBot, ClaudeBot, PerplexityBot, Google-Extended) are allowed |
| `href="#"` usage | Count of anchor links with non-functional `href` attributes |
| Generic link text | Count of links with text like "click here," "read more," "learn more" without descriptive context |
| JavaScript-gated navigation | Navigation paths that require JS execution to access (hamburger menus without `<a>` fallbacks) |

**Scoring bands:**

| Score | Condition |
|-------|-----------|
| 90–100 | XML sitemap, breadcrumbs, 100% static links, ≤3 click depth, AI crawlers allowed |
| 70–89 | Sitemap present, mostly static links, some JS-enhanced nav, minor depth issues |
| 50–69 | No sitemap, mixed static/JS navigation, moderate depth, some AI crawlers blocked |
| 25–49 | No sitemap, significant JS-gated navigation, AI crawlers partially blocked, deep link structures |
| 0–24 | No sitemap, primarily JS-only navigation, AI crawlers blocked, infinite scroll only |

### 3.6 Agent Response Fitness (Weight: 10%)

**Core question:** When the site's content is read as plain text — stripped of all visual formatting — does it tell a coherent, useful story that an agent can summarize accurately?

**Why it matters:** Ultimately, an agent consuming a site needs to produce a useful summary, comparison, or recommendation for the user it serves. If the site's information architecture relies on visual hierarchy (large text = important, red text = urgent, spatial proximity = related), those signals are lost when an agent reads content as a text stream.

**Measurement criteria:**

| Signal | Method |
|--------|--------|
| Plain-text coherence | Read the page's text content in DOM order — does it form a logical narrative? |
| Heading descriptiveness | Do headings accurately summarize the section content that follows? |
| Alt text quality | Are informational images described with meaningful, context-appropriate alt text? |
| Link text quality | Are links descriptive ("View pricing plans") vs. generic ("click here")? |
| Content ordering | Does the most important business information appear early in the DOM? |
| Boilerplate ratio | How much of the text stream is cookie banners, disclaimers, or repeated chrome vs. actual content? |
| Visual-only information | Are status indicators, categories, or relationships conveyed via text/attributes, not just color, size, or position? |
| Duplicate content | Is heading or CTA text duplicated in the DOM (e.g., `aria-label` repeating visible text unnecessarily)? |

**Scoring bands:**

| Score | Condition |
|-------|-----------|
| 90–100 | Content reads as a clear document, descriptive headings and links, low boilerplate, information-first ordering |
| 70–89 | Mostly coherent, minor gaps where visual context is assumed, acceptable boilerplate levels |
| 50–69 | Partial coherence, some reliance on visual layout for meaning, moderate boilerplate |
| 25–49 | Fragmented, significant visual-dependency, high boilerplate ratio |
| 0–24 | Incoherent text stream, dominated by boilerplate, critical information absent or buried |

---

## 4. Scoring Methodology

### 4.1 Per-Dimension Scoring

Each dimension is scored on a 0–100 scale using the measurement criteria and scoring bands defined above. The score reflects the aggregate assessment across all measured signals within the dimension.

Individual findings within each dimension are classified by severity:

| Severity | Definition |
|----------|-----------|
| Critical | Renders the dimension effectively non-functional for agents. Immediate remediation required. |
| High | Significant barrier to agent interaction. Should be addressed in the near term. |
| Medium | Measurable negative impact on agent experience. Recommended improvement. |
| Low | Minor friction or missed optimization. Address when convenient. |
| Info | Observation with no direct impact. Context for future improvement. |

### 4.2 Composite ASM Score

The composite ASM score is a weighted average of the six dimension scores. Weights reflect the dependency chain — a site that fails on Content Survivability can't score well on anything downstream.

| Dimension | Weight | Rationale |
|-----------|--------|-----------|
| Content Survivability | 25% | Without visible content, nothing else matters. |
| Structural Legibility | 20% | Agents must understand page structure to navigate and act. |
| Interactive Manifest Clarity | 20% | Action invocation is the core value proposition of agent interaction. |
| Data Extractability | 15% | Critical for comparison, evaluation, and recommendation agents. |
| Navigation Traversability | 10% | Affects discovery, but agents often arrive at specific pages via external search. |
| Agent Response Fitness | 10% | Important for summary quality but less impactful than functional access. |

**Composite = (CS × 0.25) + (SL × 0.20) + (IMC × 0.20) + (DE × 0.15) + (NT × 0.10) + (ARF × 0.10)**

### 4.3 Grade Scale

| Composite | Grade | Interpretation |
|-----------|-------|----------------|
| 90–100 | A | **Agent-Ready.** The site is optimized for the agentic web. |
| 75–89 | B | **Agent-Functional.** Minor improvements recommended. |
| 60–74 | C | **Agent-Impaired.** Notable gaps in agent capability. Agents can partially operate the site but will fail on key interactions. |
| 40–59 | D | **Agent-Hostile.** Significant remediation required. Most agent interactions will fail. |
| 0–39 | F | **Agent-Invisible.** Fundamental architectural changes needed. Agents cannot meaningfully operate this site. |

### 4.4 Score Presentation

A standard ASM scorecard presents:

1. **Composite ASM score** with letter grade
2. **Per-dimension scores** with pass/warn/fail indicators (pass ≥ 70, warn 40–69, fail < 40)
3. **Key diagnostic metrics** displayed inline: JS dependency %, semantic HTML ratio %, static link %
4. **Finding count by severity** (critical / high / medium / low / info)
5. **Top 5 priority actions** ranked by impact

---

## 5. What Accessibility Doesn't Cover

There is real overlap between web accessibility best practices and ASM readiness. Both rely on semantic HTML and the accessibility tree. Organizations that have invested in WCAG compliance have a genuine head start on agent-readiness.

**A WCAG-compliant site with server-side rendering will likely score in the B or C range on ASM without any additional work.** That's not nothing — it means the foundation is solid. Semantic landmarks, heading hierarchy, accessible names on controls, descriptive alt text — all of this serves agents as well as it serves humans with assistive technology.

But the head start plateaus. WCAG doesn't measure the things that separate a B from an A: structured data extraction, agent traffic governance, manifest-level site orientation, content survivability without JavaScript, and machine-readable action contracts. ASM is the last mile — the delta between "agents can partially use this site" and "agents can fully operate this site."

### 5.1 Different Consumers, Different Needs

A screen reader mediates between a machine representation and a human brain. It translates structure into narration. The human on the other end has context, patience, and the ability to interpret ambiguity.

An agent is the final consumer. There is no human interpreting its output in real-time. It needs structured, unambiguous, machine-parseable data. It doesn't need narration — it needs an API.

The screen reader says: *"Button: Add Apex Runner V3 to cart, $149.00, in stock."*

The agent needs: `product_id: apex-runner-v3`, `price: 149.00`, `currency: USD`, `availability: in_stock`, and an invocable action target that will execute the purchase.

Same page. Same DOM. Fundamentally different consumption models.

### 5.2 What WCAG Tests vs. What Agents Need

| Capability | WCAG Catches It | ASM Catches It |
|-----------|:-:|:-:|
| Semantic HTML landmarks | ✓ | ✓ |
| Heading hierarchy | ✓ | ✓ |
| Alt text on images | ✓ | ✓ |
| Native interactive elements | ✓ | ✓ |
| Accessible names on controls | ✓ | ✓ |
| Keyboard operability | ✓ | ✓ |
| Descriptive link text | ✓ | ✓ |
| **Content without JavaScript** | — | ✓ |
| **robots.txt AI crawler access** | — | ✓ |
| **schema.org structured data** | — | ✓ |
| **Consistent DOM patterns** | — | ✓ |
| **Pagination vs. infinite scroll** | — | ✓ |
| **Link depth analysis** | — | ✓ |
| **Boilerplate ratio** | — | ✓ |
| **Machine-readable data values** | — | ✓ |
| **Agent access policy (three-tier model)** | — | ✓ |
| **Agent response fitness (text stream coherence)** | — | ✓ |
| **Visual-only state indicators** | Partially | ✓ |
| **Data extractability (pricing, specs, inventory)** | — | ✓ |

WCAG compliance covers roughly 40% of what ASM measures. The 60% it doesn't cover — Content Survivability, Data Extractability, Navigation Traversability, Agent Response Fitness, and the three-tier access model — is where agents actually fail.

### 5.3 The Critical Divergence: JavaScript

This is the single most important difference, and it's the one that kills the "we already do this" assumption.

Accessibility testing is performed with JavaScript enabled. Screen readers operate on the fully rendered page. Every accessibility auditor in the world tests with JS running. There is no WCAG criterion that evaluates what happens when JavaScript is disabled.

Agents frequently operate without JavaScript. A significant portion of agent traffic issues a raw GET request, parses the HTML response, and never executes a script. It's faster, cheaper, more reliable, and more secure.

**A fully WCAG-compliant site can score zero on Content Survivability.**

A React single-page application with perfect accessibility — every ARIA attribute in place, every heading in sequence, every button labeled, every contrast ratio passing — returns an empty `<div id="app"></div>` to an agent that doesn't execute JavaScript. The agent sees nothing. Not a single word of content. Not a single interactive element. The entire site is invisible.

This is not a theoretical edge case. It describes the majority of modern web applications built on React, Vue, and Angular without server-side rendering.

### 5.4 The Screen Reader vs. Agent Conflict

Screen readers and agents have opposing needs in some areas:

A screen reader benefits from verbose, contextual descriptions. "Image: A pair of blue and white Apex Runner V3 running shoes, shown from a three-quarter angle, featuring mesh upper and responsive cushioning sole" helps a visually impaired user understand the product.

An agent needs: `product: Apex Runner V3`, `category: running shoes`, `color: blue/white`. The verbose description is noise that the agent has to parse through to extract the structured data it actually needs.

Accessible names on controls help both consumers, but the *information density* they need is different. A screen reader user benefits from "Add Apex Runner V3 to cart — currently $149.00, in stock, free shipping over $75." An agent benefits from discrete, labeled data fields it can extract without natural language parsing.

WCAG optimizes for human comprehension through assistive technology. ASM optimizes for machine comprehension without any assistive translation layer.

---

## 6. What ASM Is Not

ASM occupies a specific gap that existing standards don't address. Understanding what ASM *is not* prevents the most common misreadings.

| Standard | What It Solves | What It Doesn't Solve |
|----------|---------------|----------------------|
| **MCP** | How an LLM connects to external tools | Whether your page is readable when fetched |
| **A2A** | How agents communicate with each other | Whether your page is navigable by an agent |
| **AGENTS.md** | Developer-facing agent instructions in markdown | Machine-readable governance and traffic policy |
| **GEO** | Whether AI models cite your brand in answers | Whether agents can operate and transact on your site |
| **WCAG** | Whether humans with assistive tech can use your site | Whether machines with no human in the loop can use your site |

Every standard above is valuable. None of them answer the question ASM answers: **when an agent fetches your page, can it find the content, understand the structure, locate the actions, and extract the data?**

MCP defines how agents connect to tools — it says nothing about what happens when those tools fetch a web page. A2A defines how agents talk to each other — it says nothing about whether your site is legible to either of them. AGENTS.md provides prose guidance for developers building agents — it doesn't give agents themselves a machine-readable manifest to consume. GEO optimizes for whether AI *mentions* your brand — ASM optimizes for whether agents can *use* your site. WCAG ensures humans with assistive technology can access your content — ASM ensures machines with no human in the loop can operate it.

ASM is infrastructure, not marketing. It's plumbing, not reputation. It answers: can the agent see the content, understand the structure, find the buttons, and extract the data?

---

## 7. Common Failure Patterns

These patterns are diagnostic — they describe what ASM measurement reveals, not how to fix it. Remediation guidance is defined in the ASM Build Spec.

### 7.1 The SPA Blank Page

**Dimension:** Content Survivability (critical failure)
**Pattern:** Site is a React/Vue/Angular SPA that renders entirely client-side. Initial HTML response contains only a root `<div>` and script tags.
**ASM signal:** Text content ratio (no-JS vs. full render) < 5%.
**Prevalence:** Extremely common in modern web applications.

**What the agent sees:**

```
GET https://example.com/products/apex-runner-v3

Response body:
<html><head><title></title></head>
<body><div id="app"></div>
<script src="/bundle.js"></script></body></html>

Agent extraction: [empty — no content, no product, no price, no actions]
```

### 7.2 Div Soup

**Dimension:** Structural Legibility (poor to critical)
**Pattern:** Page layout built entirely with `<div>` elements differentiated only by CSS classes. No semantic landmarks, no heading hierarchy, no structural signals in the DOM.
**ASM signal:** Semantic-to-generic ratio < 3%. Zero landmarks detected.

### 7.3 The Invisible CTA

**Dimension:** Interactive Manifest Clarity (critical failure)
**Pattern:** Primary call-to-action is a styled `<div>` or `<span>` with a JavaScript click handler. Appears as a button to humans but doesn't exist in the accessibility tree as an interactive element.
**ASM signal:** Primary CTA absent from accessibility tree. Generic clickable element count > 0.

**What the agent sees:**

```
Rendered page (human view): [Buy Now] button, green, centered, prominent
Accessibility tree (agent view): ... (no interactive element found) ...

Agent action: Cannot identify purchase action. Skipping site.
```

### 7.4 Data in Images

**Dimension:** Data Extractability (critical failure)
**Pattern:** Pricing tables, comparison charts, or product specifications rendered as images or infographics with no HTML text equivalent.
**ASM signal:** Critical business data absent from DOM text content. Image-heavy sections with no alt text or non-descriptive alt text.

### 7.5 JavaScript-Gated Navigation

**Dimension:** Navigation Traversability (poor to critical)
**Pattern:** Primary navigation requires JavaScript to open (hamburger menus with no `<a>` fallbacks), uses infinite scroll instead of pagination, or relies on JS routing with no static URL structure.
**ASM signal:** Static link coverage < 50%. Navigation paths require JS execution to access.

### 7.6 The Locked Door

**Dimension:** Navigation Traversability (high)
**Pattern:** `robots.txt` blocks AI crawlers (GPTBot, ClaudeBot, PerplexityBot, Google-Extended) — often from default CMS configurations the site owner never reviewed. Note: current AI user-agent strings do not distinguish between crawlers and agents. OpenAI's Operator (a user-directed agent) uses the same `GPTBot` header as OpenAI's training crawler. Blocking `GPTBot` in `robots.txt` blocks both the crawler you don't want and the customer agent you do want.
**ASM signal:** AI crawler access check returns blocked. Site owner unaware.

### 7.7 Visual-Dependent Information

**Dimension:** Agent Response Fitness (poor)
**Pattern:** Important information conveyed through color alone (red = out of stock, green = available), spatial relationships (related items positioned near each other with no semantic grouping), or visual size (large text = important).
**ASM signal:** Visual-only information indicators detected. Status/state information absent from DOM text and attributes.

### 7.8 The Incoherent Stream

**Dimension:** Agent Response Fitness (poor)
**Pattern:** Page content, when read as linear text, is dominated by cookie banners, promotional overlays, repeated navigation chrome, and boilerplate. Actual business content is buried or fragmented.
**ASM signal:** Boilerplate ratio > 40%. Business-purpose identification requires reading past the first 500 words.

---

## 8. What Agents Actually See — Before and After

The following example shows the same product page in two implementations. Both are visually identical to a human visitor. Both pass WCAG AA. The difference is what an agent can extract and act on.

### Before: Typical Implementation

```html
<div class="product-card">
  <div class="product-image">
    <img src="shoe.jpg">
  </div>
  <div class="product-info">
    <div class="product-title">Apex Runner V3</div>
    <div class="product-price">$149.00</div>
    <div class="stock-badge green">●</div>
    <div class="btn btn-primary" onclick="addToCart(12345)">ADD TO CART</div>
  </div>
</div>
```

**What the agent extracts:**

```
container_text: "Apex Runner V3 $149.00 ● ADD TO CART"
structured_data: none
interactive_elements: none (div with onclick is invisible to accessibility tree)
product_id: unknown
price_value: unknown (requires NLP to parse "$149.00" from unstructured text)
availability: unknown (green dot conveys nothing without visual rendering)
purchase_action: not found
```

**Agent result:** Cannot reliably identify product, price, availability, or purchase action. Site skipped.

### After: Agent-Ready Implementation

```html
<article itemscope itemtype="https://schema.org/Product">
  <img src="shoe.jpg" alt="Apex Runner V3 running shoes, blue and white mesh upper">
  <h2 itemprop="name">Apex Runner V3</h2>
  <p itemprop="offers" itemscope itemtype="https://schema.org/Offer">
    <span itemprop="price" content="149.00">$149.00</span>
    <meta itemprop="priceCurrency" content="USD">
    <link itemprop="availability" href="https://schema.org/InStock">
    <span data-asm-state="in-stock" data-asm-state-label="In stock — ships in 2 business days">
      In Stock
    </span>
  </p>
  <button data-asm-action="add-to-cart"
          data-asm-target="product:apex-runner-v3"
          data-asm-consequences="Item added to cart. Cart count increments. Cart drawer opens."
          aria-label="Add Apex Runner V3 to cart">
    Add to Cart
  </button>
</article>
```

**What the agent extracts:**

```
product_name: "Apex Runner V3"
product_type: schema.org/Product
price: 149.00
currency: USD
availability: InStock (schema.org/InStock)
availability_detail: "In stock — ships in 2 business days"
purchase_action: button[data-asm-action="add-to-cart"]
action_target: product:apex-runner-v3
action_consequences: "Item added to cart. Cart count increments. Cart drawer opens."
```

**Agent result:** Full product data extracted. Purchase action identified and invocable. Consequences understood. Transaction can proceed.

Same product. Same visual presentation. Completely different agent capability.

---

## 9. Conducting an ASM Audit

### 9.1 Audit Scope

An ASM audit can be conducted at three levels:

| Level | Scope | Use Case |
|-------|-------|----------|
| **Page-level** | Homepage + 2–5 key pages | Initial assessment, focused remediation |
| **Section-level** | A functional area (e-commerce catalog, blog, support docs) | Representative score for a content type |
| **Site-level** | All major templates and content types | Definitive ASM score, full remediation roadmap |

### 9.2 Audit Deliverables

A standard ASM audit produces:

1. **ASM Scorecard** — Per-dimension scores and composite score with grade.
2. **Findings Report** — Specific issues identified in each dimension with severity ratings.
3. **Priority Remediation List** — Top actions ranked by impact × effort. (Detailed remediation patterns reference the ASM Build Spec.)

### 9.3 Tooling Requirements

ASM measurement requires:

- A browser automation environment capable of rendering pages with and without JavaScript
- Access to the browser's accessibility tree (e.g., via CDP `Accessibility.getFullAXTree`)
- DOM introspection for element counting, attribute inspection, and structural analysis
- Network-level inspection for resource loading analysis
- Multi-viewport testing capability

The framework is tool-agnostic. Any implementation that can perform the measurements defined in Section 3 can produce valid ASM scores.

---

## 10. Relationship to ASM Build Spec

The ASM Scoring Framework defines *what* to measure. The ASM Build Spec defines *how to build for it*.

| | ASM Scoring Framework | ASM Build Spec |
|--|-----|-------|
| **Purpose** | Diagnostic scoring | Implementation standard |
| **Audience** | Site owners, auditors, consultants | Developers, framework authors, CMS builders |
| **Output** | Scores, findings, gap analysis | Manifest spec, markup patterns, compliance levels |
| **Question** | "How agent-ready is this site?" | "How do I build an agent-ready site?" |

The two are designed as a diagnostic-and-treatment pair. ASM scores identify gaps. ASM patterns close them.

See: **ASM Build Spec** (separate document)

---

## Versioning

ASM is a living framework. As AI agent capabilities evolve, the dimensions, measurement criteria, and scoring bands will be updated.

- **v1.0 (March 2026):** Initial release. Six dimensions, quantitative scoring bands, failure patterns, before/after examples. Companion to ASM Build Spec v1.0.
- **Future versions** will incorporate findings from real-world audits, community feedback, and adaptation to new agent capabilities and protocols (MCP, A2A, ACP, and successors).

Audit reports reference the framework version used.

---

## License and Attribution

The ASM Scoring Framework is developed and maintained by Wesley Shoffner at Clocktower & Associates.

The framework is published for industry adoption. Organizations conducting ASM audits are encouraged to reference this specification and contribute feedback to its evolution.

For audit services, consulting, or ASM implementation support: [Clocktower & Associates](https://www.clocktowerassoc.com)
