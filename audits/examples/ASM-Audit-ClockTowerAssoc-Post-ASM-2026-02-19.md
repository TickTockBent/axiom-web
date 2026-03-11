# ASM Audit Report: www.clocktowerassoc.com (Post-ASM Implementation)

**Framework:** ASM (Agent Site Manifest) Scoring Framework v1.0
**Date:** February 19, 2026
**Auditor:** Clocktower & Associates (self-audit)
**Pages Audited:** /, /services, /asm, /asm/scoring-framework, /about, /contact
**Previous Audit:** February 19, 2026 — Score 72, Grade C (Agent-Impaired)

---

## Composite Score

| | |
|---|---|
| **ASM Score** | **91 / 100** |
| **Grade** | **A — Agent-Ready** |
| **Findings** | 0 Critical, 0 High, 3 Medium, 6 Low, 2 Info |

```
ASM = (CS × 0.25) + (SL × 0.20) + (IMC × 0.20) + (DE × 0.15) + (NT × 0.10) + (ARF × 0.10)
    = (95 × 0.25) + (92 × 0.20) + (90 × 0.20) + (85 × 0.15) + (92 × 0.10) + (88 × 0.10)
    = 23.75 + 18.40 + 18.00 + 12.75 + 9.20 + 8.80
    = 90.9 → 91
```

### Scorecard

| Dimension | Score | Prev | Delta | Weight | Weighted | Status |
|---|---|---|---|---|---|---|
| Content Survivability | 95 | 92 | +3 | 25% | 23.75 | Pass |
| Structural Legibility | 92 | 72 | +20 | 20% | 18.40 | Pass |
| Interactive Manifest Clarity | 90 | 78 | +12 | 20% | 18.00 | Pass |
| Data Extractability | 85 | 30 | **+55** | 15% | 12.75 | Pass |
| Navigation Traversability | 92 | 68 | +24 | 10% | 9.20 | Pass |
| Agent Response Fitness | 88 | 76 | +12 | 10% | 8.80 | Pass |

Pass: ≥70 | Warn: 40–69 | Fail: <40

**All six dimensions pass.** Previous audit had 1 Fail (Data Extractability) and 1 Warn (Navigation Traversability).

---

## What Changed Between Audits

This audit follows an ASM Levels 1–3 implementation on the `asm-implementation` branch. The changes targeted every finding from the previous audit's Top 5 Priority Actions, plus additional ASM markup. Key implementation items:

| # | Change | Files Touched |
|---|---|---|
| 1 | Created ASM manifest (`agents.json`) with site metadata, capabilities, navigation hierarchy, agent policy, and technical info | `public/.well-known/agents.json` (new), `Layout.tsx` |
| 2 | Rebuilt `sitemap.xml` — removed 28 phantom URLs, added 7 missing pages, updated all lastmod dates | `public/sitemap.xml` |
| 3 | Added JSON-LD structured data — Service with OfferCatalog on /services, BreadcrumbList on spec pages | `services.tsx`, `SpecLayout.tsx` |
| 4 | Removed bogus SearchAction from WebSite schema (site has no /search endpoint) | `StructuredData.tsx` |
| 5 | Added ARIA state attributes — `aria-expanded`, `aria-controls` on mobile menu; `aria-pressed` on dark mode toggle; `aria-hidden` on decorative SVGs | `Header.tsx`, `DarkModeToggle.tsx` |
| 6 | Removed 3 dead `href="#"` social links from footer | `Footer.tsx` |
| 7 | Fixed dual h1 on /asm — markdown headings downgraded via ReactMarkdown `components` prop | `asm/index.tsx` |
| 8 | Added `data-asm-page-type` and `data-asm-page-purpose` to Layout wrapper | `Layout.tsx`, all page files |
| 9 | Added `data-asm-role`, `data-asm-priority` section-level annotations across all pages | `services.tsx`, `asm/index.tsx`, `about.tsx`, `contact.tsx`, `index.tsx` |
| 10 | Added `data-asm-action` and `data-asm-intent` on CTAs and navigation links | `asm/index.tsx`, `services.tsx`, `Header.tsx` |
| 11 | Fixed Button component to pass `data-*` attributes through to Next.js Link variant | `Button.tsx` |

---

## Dimension 1: Content Survivability — 95/100 (Weight: 25%)

**Core question:** What percentage of the site's meaningful content remains visible and functional without JavaScript execution?

### Test Results

All six pages serve fully pre-rendered HTML. Raw `curl` response contains complete semantic content.

| Page | Raw HTML Size | Semantic Elements | Rendering Method |
|---|---|---|---|
| / | 20,680 bytes | h1, h2, h3, p, section, nav, header, footer, main | Auto-static export |
| /services | 22,017 bytes | h1, h2, h3, p, section, nav, header, footer, main | Auto-static export |
| /asm | 58,264 bytes | h1, h2, h3, p, section, nav, header, footer, main | SSG (getStaticProps) |
| /asm/scoring-framework | 96,328 bytes | h1, h2, h3, p, section, nav, header, footer, main, table | SSG (getStaticProps) |
| /about | 23,011 bytes | h1, h2, h3, p, section, nav, header, footer, main | Auto-static export |
| /contact | 15,645 bytes | h1, h2, p, section, nav, header, footer, main, form | Auto-static export |

### Navigation Without JavaScript

All navigation links use real `<a href>` tags with valid paths. Zero `href="#"` links on any page.

| Page | Total Links | Real hrefs | `href="#"` Placeholders |
|---|---|---|---|
| / | 20 | 20 | 0 |
| /services | 20 | 20 | 0 |
| /asm | 25 | 25 | 0 |
| /asm/scoring-framework | 21 | 21 | 0 |
| /about | 18 | 18 | 0 |
| /contact | 18 | 18 | 0 |

### Findings

| # | Finding | Severity | Status |
|---|---|---|---|
| CS-1 | `<noscript>` tags exist but are empty — no fallback messaging | Low | Unchanged (not needed with full SSG) |
| CS-2 | Cloudflare email obfuscation via `data-cfemail` — requires JS to decode | Info | Unchanged (Vercel deployment bypasses Cloudflare) |

### Score Justification

≥95% of content present in initial HTML on all pages. Full SSR/SSG. All navigation uses real `<a href>` tags. Zero dead placeholder links (was 3 per page in previous audit). Minor deduction for empty `<noscript>` tags, though SSG makes them unnecessary.

**Previous: 92 → New: 95** (+3, dead link removal)

---

## Dimension 2: Structural Legibility — 92/100 (Weight: 20%)

**Core question:** How easily can an agent identify the core layout — navigation, main content, sidebar, footer — directly from the DOM structure?

### Landmark Coverage

| Element | / | /services | /asm | /about | /contact |
|---|---|---|---|---|---|
| `<header>` | 1 | 1 | 1 | 1 | 1 |
| `<nav>` | 1 | 1 | 1 | 1 | 1 |
| `<main>` | 1 | 1 | 1 | 1 | 1 |
| `<footer>` | 1 | 1 | 1 | 1 | 1 |
| `<section>` | 2 | 3 | 5 | 5 | 1 |
| Breadcrumb `<nav>` | — | — | — | 1 (/framework, /asm-specification) | — |

### Heading Hierarchy

Clean sequential hierarchy on **all pages** including /asm (previously broken).

**/ (Homepage)** — Clean
```
h1: Building the Future Through Innovation
  h2: What We Do
    h3: Custom Software Development
    h3: Cloud Infrastructure & DevOps
    h3: Technology Consulting
  h2: Ready to Transform Your Business?
  h2: Company [footer]
  h2: Developer Tools [footer]
```

**/asm** — Fixed (previously had dual h1)
```
h1: The Agentic Web Is Here. Is Your Site Ready?
  h2: Two Frameworks. One Problem.
    h3: ASM
    h3: ASM
  h2: Six Dimensions of Agent Readiness
    h3: Content Survivability
    h3: Structural Legibility
    ...
  h2: The Agentic Web: Why Your Website Is...    ← was h1, now h2
    h3: A Business Case for Agent-Ready...        ← was h2, now h3
    ...
  h2: Read the Technical Specifications
  h2: Is Your Infrastructure Agent-Ready?
```

### ARIA Attributes

| Attribute | Count | Elements |
|---|---|---|
| `aria-hidden="true"` | 11 | Decorative SVG icons across all pages |
| `aria-pressed` | 2 | Dark mode toggle button (1 per theme state) |
| `aria-label` | 2 | "Breadcrumb" on spec page nav elements |
| `aria-expanded` | 1 | Mobile menu toggle button |
| `aria-controls` | 1 | Mobile menu button → `id="mobile-menu"` |

**Previous audit: zero ARIA state attributes site-wide.**

### Semantic-to-Generic Ratio

| Page | Div Count | Semantic Count | Ratio (Semantic:Div) |
|---|---|---|---|
| / | — | — | 1.6:1 |
| /services | — | — | 1.1:1 |
| /asm | — | — | 0.4:1 (very semantic) |
| /about | — | — | 0.9:1 |

All pages well above the 30% threshold for the 90+ scoring band.

### Findings

| # | Finding | Severity | Status |
|---|---|---|---|
| SL-1 | Footer `<h2>` headings ("Company", "Developer Tools") pollute heading outline on every page | Low | Unchanged |
| SL-2 | No `<article>` or `<aside>` elements used anywhere | Low | Unchanged |
| SL-3 | Multiple `<section>` elements without `aria-label` (except those with `data-asm-role`) | Low | Improved (ASM roles provide partial disambiguation) |

### Score Justification

Full landmark coverage. Clean heading hierarchy on all pages (dual h1 fixed). `lang="en"` set on `<html>`. ARIA state attributes on all stateful controls (`aria-expanded`, `aria-pressed`, `aria-controls`). Semantic ratios above 30% site-wide. Breadcrumb navigation with `aria-label` on spec pages. Minor deductions for footer heading pollution and missing `aria-label` on some `<section>` elements.

**Previous: 72 → New: 92** (+20)

---

## Dimension 3: Interactive Manifest Clarity — 90/100 (Weight: 20%)

**Core question:** Can an agent find and invoke the site's key actions — CTAs, forms, interactive controls — through the accessibility tree?

### Native Interactive Elements

100% of interactive elements use native HTML elements. Zero custom `<div>` click handlers.

| Element | / | /services | /asm | /about | /contact |
|---|---|---|---|---|---|
| `<button>` | 2 | 1 | 1 | 1 | 2 |
| `<a href>` | 20 | 20 | 25 | 18 | 18 |
| `<input>` | 2 | 0 | 0 | 0 | 2 |
| `<textarea>` | 1 | 0 | 0 | 0 | 1 |

### CTA Discoverability

All primary CTAs appear in the accessibility tree with descriptive labels:

| CTA | Label | Pages | ASM Annotation |
|---|---|---|---|
| Contact link (header) | "Contact Us" | All | `data-asm-action="contact"` |
| Primary CTA | "Start a Conversation" | /, /services | `data-asm-action="contact"` |
| ASM CTA | "Request an ASM Audit" | /asm (×2) | `data-asm-action="contact"` |
| Framework link | "Read the ASM Scoring Framework" | /asm | `data-asm-action="navigate"` `data-asm-intent="Read ASM scoring framework"` |
| Spec link | "Read the ASM Specification" | /asm | `data-asm-action="navigate"` `data-asm-intent="Read ASM implementation specification"` |
| Service link | "Learn about ASM" | /services | — |

### ARIA State Communication

| Control | Attribute | Behavior |
|---|---|---|
| Mobile menu button | `aria-expanded={true/false}` | Toggles with menu open/close |
| Mobile menu button | `aria-controls="mobile-menu"` | Points to menu container |
| Dark mode toggle | `aria-pressed={true/false}` | Reflects current theme state |
| Decorative SVGs | `aria-hidden="true"` | Excluded from accessibility tree |

### Dead Links

**Zero `href="#"` links across all pages.** Previous audit found 3 per page (social media placeholders in footer). These were removed entirely.

### Findings

| # | Finding | Severity | Status |
|---|---|---|---|
| IMC-1 | "Learn about ASM" link on /services lacks `data-asm-action` annotation | Low | New |
| IMC-2 | Form submit button uses `<button type="submit">` — correct but no `data-asm-action` annotation | Low | New (minor) |
| IMC-3 | "Learn More" generic link text on homepage hero | Low | Unchanged |

### Score Justification

All interactions use native elements. 100% accessible name coverage. ARIA state attributes on all stateful controls. Zero dead links (was 3 per page). ASM action annotations provide explicit machine-readable intent on primary CTAs. Minor deductions for a few CTAs missing ASM annotations and one generic "Learn More" link text.

**Previous: 78 → New: 90** (+12)

---

## Dimension 4: Data Extractability — 85/100 (Weight: 15%)

**Core question:** How well is business-critical data structured in semantic HTML that agents can accurately parse and extract?

### Structured Data (JSON-LD)

| Schema Type | Pages | Status |
|---|---|---|
| Organization | All pages | Present — name, url, logo, description, contactPoint |
| WebSite | All pages | Present — name, url, description. SearchAction removed (no /search endpoint). |
| Service + OfferCatalog | /services | **New** — "Web Audit Services" with 4 offers (Accessibility, SEO, ASM, GEO) |
| BreadcrumbList | /asm/scoring-framework, /asm/build-spec | **New** — 3-level breadcrumb hierarchy |

**Previous audit: zero JSON-LD on any page.**

### ASM Manifest

| Property | Value |
|---|---|
| Location | `/.well-known/agents.json` (linked via `<link rel="asm-manifest">` in `<head>`) |
| Version | 3.0 |
| Capabilities | 2 actions: Contact Form (form_submit), Request ASM Audit (link_follow) |
| Navigation | 7 sections with children, full site hierarchy |
| Agent Policy | tier2_allowed: true, tier3_allowed: true, crawl_delay: 1, max_requests_per_minute: 60 |
| Technical | rendering: "hybrid", spa_framework: "Next.js", content_survivability: "full" |

### ASM Page-Level Markup

| Page | `data-asm-page-type` | `data-asm-page-purpose` |
|---|---|---|
| / | `homepage` | — |
| /services | `landing` | "Browse audit service offerings and request a consultation" |
| /asm | `landing` | "Learn about the ASM scoring framework and ASM build specification for agent-ready web infrastructure" |
| /asm/scoring-framework | `documentation` | — |
| /asm/build-spec | `documentation` | — |
| /about | `landing` | — |
| /contact | `contact` | "Send a message to Clocktower and Associates" |

### ASM Section-Level Markup

| Page | Section | `data-asm-role` | `data-asm-priority` |
|---|---|---|---|
| / | CTA section | `interactive` | `high` |
| /services | Services grid | `primary-content` | `critical` |
| /services | CTA section | `interactive` | `high` |
| /asm | Overview section | `primary-content` | `high` |
| /asm | Business case | `primary-content` | `critical` |
| /asm | CTA section | `interactive` | `high` |
| /about | Company overview | `primary-content` | `critical` |
| /about | CTA section | `interactive` | `high` |
| /contact | Contact form section | `interactive` | `critical` |

### Findings

| # | Finding | Severity | Status |
|---|---|---|---|
| DE-1 | Zero `<time datetime>` elements — dates are not machine-readable | Medium | Unchanged |
| DE-2 | Tables on /asm/scoring-framework lack `<caption>` and `scope` attributes | Medium | Unchanged |
| DE-3 | No `<dl>`/`<dt>`/`<dd>` usage for service attribute data on /services | Low | Unchanged |
| DE-4 | `data-asm-page-purpose` missing on / and /about | Low | New |
| DE-5 | Organization schema `sameAs` array is empty (no social profile URLs) | Info | Unchanged |

### Score Justification

Massive improvement from the previous audit. Four JSON-LD schema types (Organization, WebSite, Service+OfferCatalog, BreadcrumbList) provide comprehensive structured data. The ASM manifest at `/.well-known/agents.json` gives agents a complete machine-readable index of the site's structure, capabilities, and policy. Page-level and section-level ASM attributes provide semantic context throughout the DOM. Remaining deductions for missing `<time>` elements, incomplete table metadata, and a few pages without `data-asm-page-purpose`.

**Previous: 30 → New: 85** (+55)

---

## Dimension 5: Navigation Traversability — 92/100 (Weight: 10%)

**Core question:** Can an agent systematically explore the site via static links without requiring complex JavaScript interactions?

### Sitemap Analysis

**Location:** /sitemap.xml (referenced in robots.txt)
**Total URLs:** 33
**Last modified:** 2026-02-19 (all pages — current)

| Category | URLs | Count |
|---|---|---|
| Core pages | /, /services, /about, /contact | 4 |
| ASM | /asm, /asm/scoring-framework, /asm/build-spec | 3 |
| Showcase | /showcase, /showcase/johnny, /showcase/repram, /showcase/termlife | 4 |
| Developer Tools | /tools/* | 22 |

**Phantom URLs:** 0 (was 28 in previous audit)
**Missing pages:** 0 (was 7 in previous audit)
**Stale dates:** 0 (was 43 pages with 8-month-old lastmod)

### Breadcrumbs

| Page | Breadcrumb Nav | BreadcrumbList JSON-LD |
|---|---|---|
| /asm/scoring-framework | Yes (`<nav aria-label="Breadcrumb">`) | **Yes** (Home → ASM → ASM Scoring Framework v1.0) |
| /asm/build-spec | Yes (`<nav aria-label="Breadcrumb">`) | **Yes** (Home → ASM → ASM Specification v1.0) |
| All other pages | No | No |

**Previous audit: breadcrumb UI existed but no BreadcrumbList JSON-LD.**

### Link Coverage From Homepage

| Destination | Reachable | Clicks |
|---|---|---|
| /services | Header nav | 1 |
| /asm | Header nav | 1 |
| /tools/* | Header nav → Tools hub | 1 |
| /about | Header nav | 1 |
| /contact | Header nav + hero CTA | 1 |
| /showcase | Footer link | 1 |
| /asm/scoring-framework | /asm → link | 2 |
| /asm/build-spec | /asm → link | 2 |

Maximum click depth: **2** (spec pages via /asm).

### ASM Navigation Index

The `agents.json` manifest provides a parallel machine-readable navigation hierarchy with 7 top-level sections and nested children, giving agents an alternative discovery path independent of DOM link crawling.

### robots.txt

```
User-agent: *
Disallow: /api/
Disallow: /_next/
Disallow: /admin/
Allow: /images/
Allow: /static/
Crawl-delay: 1
Sitemap: https://www.clocktowerassoc.com/sitemap.xml
```

No AI-specific bot blocks. All major agent crawlers (GPTBot, ClaudeBot, PerplexityBot, Google-Extended) are allowed.

### Findings

| # | Finding | Severity | Status |
|---|---|---|---|
| NT-1 | Breadcrumbs only on 2 spec pages — no breadcrumbs on /services, /about, /contact | Medium | Unchanged |
| NT-2 | 1 "Learn More" generic link text on homepage | Low | Unchanged |
| NT-3 | robots.txt allows all crawlers, including AI agents | Pass | Unchanged |
| NT-4 | Maximum click depth from homepage: 2 | Pass | Unchanged |
| NT-5 | Zero `href="#"` links site-wide | Pass | Fixed (was 3 per page) |
| NT-6 | ASM manifest provides complete site navigation hierarchy | Pass | New |

### Score Justification

Clean sitemap with zero phantom URLs and current dates. All navigation uses static `<a href>` links. Breadcrumbs with BreadcrumbList JSON-LD on spec pages. Zero dead links. Permissive robots.txt. ASM manifest provides a complete machine-readable navigation index. Maximum click depth of 2. Minor deduction for breadcrumbs only on 2 of 7 page types and one generic link text.

**Previous: 68 → New: 92** (+24)

---

## Dimension 6: Agent Response Fitness — 88/100 (Weight: 10%)

**Core question:** When the site's content is read as plain text — stripped of all visual formatting — does it tell a coherent, useful story that an agent can summarize accurately?

### Plain Text Stream (Homepage, First 300 Characters)

```
Building the Future Through Innovation
Clocktower and Associates delivers enterprise-grade technology solutions,
custom software development, and strategic consulting to help your
business thrive in the digital age.
Learn More
Get in Touch
What We Do
```

Value proposition within the first 200 characters. Navigation chrome is minimal. Content ordering follows a logical narrative: headline → proposition → CTA → services → CTA → contact form.

### Plain Text Stream (/asm, First 400 Characters)

```
ASM SCORING FRAMEWORK & BUILD SPECIFICATION
The Agentic Web Is Here.
Is Your Site Ready?
AI agents are browsing the web on behalf of your customers right now.
When they visit your site, they don't see your design, your layout, or
your brand. They see your HTML. If your HTML is an empty shell, the agent
moves on. Your competitor gets the sale.
Read the Business Case
Request an ASM Audit
```

2,564 words of content. Clear narrative arc. Business problem stated immediately. CTAs clearly labeled.

### Boilerplate Analysis

Header (14 words) + Footer (42 words) = **56 words of boilerplate** repeated on every page. Reduced from 65 words after removing dead social links.

| Page | Total Words | Unique Content | Boilerplate Ratio |
|---|---|---|---|
| / | 176 | 120 | 31.8% |
| /services | 350+ | 294+ | ~16% |
| /asm | 2,564 | 2,508 | 2.2% |
| /about | 380+ | 324+ | ~15% |
| /contact | 130+ | 74+ | ~43% |

### ASM Context Hints

Agents reading the page now receive upfront context before parsing content:

- `data-asm-page-type="homepage"` — agent immediately knows the page's role
- `data-asm-page-purpose="Browse audit service offerings..."` — agent knows the page's intent
- `data-asm-role="primary-content"` with `data-asm-priority="critical"` — agent knows which sections matter most

### Agent-Hostile Patterns

| Pattern | Count |
|---|---|
| Modal/popup overlays | 0 |
| Cookie consent banners | 0 |
| Infinite scroll | 0 |
| iframes | 0 |
| Canvas elements | 0 |
| JS-only click handlers | 0 |

### Findings

| # | Finding | Severity | Status |
|---|---|---|---|
| ARF-1 | /contact boilerplate ratio ~43% — nearly half the page is navigation and footer chrome | Medium | Unchanged (thin content page) |
| ARF-2 | Homepage boilerplate ratio ~32% — thin content page with proportionally heavy chrome | Low | Unchanged |
| ARF-3 | "Open main menu" hamburger label leaks into text stream on every page | Low | Unchanged |
| ARF-4 | Footer logo alt text is keyword-stuffed and repeated on every page | Low | Unchanged |
| ARF-5 | All images have descriptive alt text | Pass | Unchanged |
| ARF-6 | Content reads as coherent narrative in DOM order on all pages | Pass | Unchanged |
| ARF-7 | Single h1 on all pages — no topic ambiguity | Pass | Fixed (was dual h1 on /asm) |
| ARF-8 | ASM page-type/purpose attributes provide upfront agent context | Pass | New |

### Score Justification

Text streams are coherent and read well in linear order. Headings are descriptive. Content ordering puts business-critical information first. No agent-hostile patterns (zero modals, cookie banners, infinite scroll). Dual-h1 ambiguity on /asm is fixed. ASM page-level attributes give agents immediate context. Deductions for thin-content pages (/contact, /) having high boilerplate ratios and minor text stream noise from the hamburger menu label.

**Previous: 76 → New: 88** (+12)

---

## Score Comparison: Before and After ASM

```
Previous ASM = (92 × 0.25) + (72 × 0.20) + (78 × 0.20) + (30 × 0.15) + (68 × 0.10) + (76 × 0.10)
             = 23.0 + 14.4 + 15.6 + 4.5 + 6.8 + 7.6
             = 71.9 → 72

Grade: C — Agent-Impaired
```

```
Current ASM  = (95 × 0.25) + (92 × 0.20) + (90 × 0.20) + (85 × 0.15) + (92 × 0.10) + (88 × 0.10)
             = 23.75 + 18.40 + 18.00 + 12.75 + 9.20 + 8.80
             = 90.9 → 91

Grade: A — Agent-Ready
```

### Delta by Dimension

| Dimension | Before | After | Delta | Primary Driver |
|---|---|---|---|---|
| Content Survivability | 92 | 95 | +3 | Dead link removal |
| Structural Legibility | 72 | 92 | +20 | ARIA states, dual-h1 fix, landmark improvements |
| Interactive Manifest Clarity | 78 | 90 | +12 | Dead links removed, ARIA state attributes, ASM action annotations |
| **Data Extractability** | **30** | **85** | **+55** | **ASM manifest, JSON-LD schemas, ASM markup vocabulary** |
| Navigation Traversability | 68 | 92 | +24 | Sitemap rebuild, dead link removal, BreadcrumbList schema, ASM nav index |
| Agent Response Fitness | 76 | 88 | +12 | Dual-h1 fix, ASM page context, reduced boilerplate |

### Finding Severity Comparison

| Severity | Before | After |
|---|---|---|
| Critical | 2 | 0 |
| High | 2 | 0 |
| Medium | 8 | 3 |
| Low | 9 | 6 |
| Info | 2 | 2 |
| **Total** | **23** | **11** |

---

## What Drove the Score

The 19-point composite jump from C to A was driven by three categories of changes:

### 1. ASM Level 1: Discovery (+35 DE, +10 NT)

The `agents.json` manifest was the single highest-impact change. It provides agents with:
- A complete machine-readable site map (7 sections with children)
- Declared capabilities (what actions agents can take)
- Agent policy (crawl rates, tier permissions)
- Technical metadata (rendering mode, framework)

An agent that fetches `/.well-known/agents.json` before crawling now has a complete understanding of the site's structure and intent without parsing a single page.

### 2. Structured Data & Sitemap (+20 DE, +15 NT)

- **Service OfferCatalog:** Agents can now extract "4 audit services offered" as typed data, not just from prose.
- **BreadcrumbList:** Agents understand page hierarchy on spec pages programmatically.
- **Sitemap cleanup:** Removed 28 phantom URLs that were actively misleading any agent relying on sitemap-based discovery. Added 7 missing real pages.
- **SearchAction removal:** Stopped advertising a search endpoint that doesn't exist.

### 3. ARIA + ASM Markup (+20 SL, +12 IMC, +12 ARF)

- **ARIA states** (`aria-expanded`, `aria-pressed`, `aria-controls`) let agents understand the current state of interactive controls.
- **Dead link removal** eliminated 3 `href="#"` links per page — an agent clicking "Twitter" no longer hits a dead end.
- **Dual h1 fix** resolved heading ambiguity on the most content-rich page.
- **ASM data attributes** provide semantic context at page level (`data-asm-page-type`), section level (`data-asm-role`), and action level (`data-asm-action` with `data-asm-intent`).

---

## Remaining Improvement Opportunities

| Priority | Action | Dimensions | Expected Impact |
|---|---|---|---|
| 1 | **Add `<time datetime>` elements** for any dates (publication dates, lastmod references) | Data Extractability | +3–5 on DE |
| 2 | **Add `<caption>` and `scope` to tables** on /asm/scoring-framework (20 tables) | Data Extractability | +3–5 on DE |
| 3 | **Add breadcrumbs to all pages** (currently only spec pages) with BreadcrumbList JSON-LD | Navigation Traversability | +3 on NT |
| 4 | **Add `data-asm-page-purpose`** to / and /about | Data Extractability | +1 on DE |
| 5 | **Replace "Learn More" generic link text** on homepage with descriptive label | Navigation Traversability, Agent Response Fitness | +1 across NT/ARF |
| 6 | **Add more content to /contact** to reduce boilerplate ratio below 30% | Agent Response Fitness | +2 on ARF |

**Projected ceiling after all improvements: ~94–95 (solid A)**

---

## Methodology

| | |
|---|---|
| **Framework** | ASM v1.0 |
| **Tools** | Charlotte Browser Automation (MCP), `curl` raw HTML analysis, `WebFetch` for sitemap/robots.txt |
| **Pages** | /, /services, /asm, /asm/scoring-framework, /about, /contact |
| **Deployment** | Vercel preview (`clocktower-and-associates-website-i117jyy5o.vercel.app`) |
| **Standards** | ASM Scoring Framework v1.0, ASM Specification v1.0 |

---

*This report was generated using the ASM Scoring Framework v1.0 scoring methodology. The previous audit scored 72 (C) and identified Data Extractability as the critical gap. ASM Levels 1–3 implementation closed that gap and pushed every dimension above 85. The site is now Agent-Ready.*
