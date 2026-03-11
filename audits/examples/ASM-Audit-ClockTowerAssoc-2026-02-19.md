# ASM Audit Report: www.clocktowerassoc.com

**Framework:** ASM (Agent Site Manifest) Scoring Framework v1.0
**Date:** February 19, 2026
**Auditor:** Clocktower & Associates (self-audit)
**Pages Audited:** /, /services, /asm, /asm/scoring-framework, /about, /contact

---

## Composite Score

| | |
|---|---|
| **ASM Score** | **72 / 100** |
| **Grade** | **C — Agent-Impaired** |
| **Findings** | 2 Critical, 2 High, 8 Medium, 9 Low, 2 Info |

```
ASM = (CS × 0.25) + (SL × 0.20) + (IMC × 0.20) + (DE × 0.15) + (NT × 0.10) + (ARF × 0.10)
    = (92 × 0.25) + (72 × 0.20) + (78 × 0.20) + (30 × 0.15) + (68 × 0.10) + (76 × 0.10)
    = 23.0 + 14.4 + 15.6 + 4.5 + 6.8 + 7.6
    = 71.9 → 72
```

### Scorecard

| Dimension | Score | Weight | Weighted | Status |
|---|---|---|---|---|
| Content Survivability | 92 | 25% | 23.0 | Pass |
| Structural Legibility | 72 | 20% | 14.4 | Pass |
| Interactive Manifest Clarity | 78 | 20% | 15.6 | Pass |
| **Data Extractability** | **30** | **15%** | **4.5** | **Fail** |
| Navigation Traversability | 68 | 10% | 6.8 | Warn |
| Agent Response Fitness | 76 | 10% | 7.6 | Pass |

Pass: ≥70 | Warn: 40–69 | Fail: <40

---

## Pre-Audit Finding: Critical Content Survivability Failure (Remediated)

Before this audit was conducted, a critical Content Survivability failure was discovered and fixed during the same session. The finding is documented here because it would have fundamentally changed the audit outcome.

### The Problem

Every page on the site was returning an empty HTML shell to non-JavaScript consumers:

```html
<html>
  <head><!-- scripts, stylesheets, meta tags --></head>
  <body>
    <div id="__next"></div>
  </body>
</html>
```

AI agents, which parse raw HTML without executing JavaScript, received zero content from every page. The site was completely invisible to the agentic web.

### Root Cause

The site's dark mode context provider wrapped the entire application in `_app.tsx` and used a common React pattern to prevent hydration mismatches:

```tsx
// DarkModeContext.tsx (before fix)
export const DarkModeProvider = ({ children }) => {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
    // ... read localStorage, set theme
  }, []);

  // This line made the entire site invisible to agents
  if (!mounted) {
    return null;
  }

  return (
    <DarkModeContext.Provider value={...}>
      {children}
    </DarkModeContext.Provider>
  );
};
```

The `mounted` state starts as `false` and only becomes `true` after `useEffect` fires — which only happens in a browser. During server-side rendering, the provider returned `null`, suppressing all page content. The build system still labeled pages as "prerendered as static content," but the HTML files contained nothing.

### The Fix

The `mounted` guard was removed. Children render unconditionally with a default state (light mode). The existing inline `DarkModeScript` in `_document.tsx` already handles the flash-of-wrong-theme problem by applying the correct CSS class before React hydrates:

```tsx
// DarkModeContext.tsx (after fix)
export const DarkModeProvider = ({ children }) => {
  const [isDarkMode, setIsDarkMode] = useState(false);

  useEffect(() => {
    // ... read localStorage, set theme (unchanged)
  }, []);

  return (
    <DarkModeContext.Provider value={...}>
      {children}
    </DarkModeContext.Provider>
  );
};
```

### Impact

| Page | Before (empty shell) | After (pre-rendered) |
|---|---|---|
| Homepage | 2.2 KB | **22.7 KB** |
| ASM Landing | 18.2 KB* | **59.5 KB** |
| Services | 2.2 KB | **22.1 KB** |

*The ASM page had `getStaticProps` data serialized in `__NEXT_DATA__`, inflating the response size despite the empty DOM.

### Score Impact

Without this fix, Content Survivability would have scored **0/100** — the textbook "SPA Blank Page" failure pattern described in Section 7.1 of the ASM Scoring Framework specification.

```
Pre-fix ASM  = (0 × 0.25) + (72 × 0.20) + (78 × 0.20) + (30 × 0.15) + (68 × 0.10) + (76 × 0.10)
             = 0 + 14.4 + 15.6 + 4.5 + 6.8 + 7.6
             = 48.9 → 49
```

**Pre-fix grade: D — Agent-Hostile.** A single line of code was the difference between Agent-Impaired and Agent-Hostile.

### Additional Access Barrier (Remediated)

During investigation, Cloudflare's managed robots.txt injection was also discovered to be prepending directives that blocked all major AI crawlers:

```
User-agent: ClaudeBot
Disallow: /

User-agent: GPTBot
Disallow: /
```

This was disabled in the Cloudflare dashboard. The site's own `robots.txt` is permissive (`User-agent: * / Allow: /`).

---

## Dimension 1: Content Survivability — 92/100 (Weight: 25%)

**Core question:** What percentage of the site's meaningful content remains visible and functional without JavaScript execution?

### Test Results

All six pages serve fully pre-rendered HTML in the raw `curl` response. The `<div id="__next">` contains complete markup on every page.

| Page | Body Word Count | `__next` Has Content | Rendering Method |
|---|---|---|---|
| / | 241 | Yes (1,820 chars) | Auto-static export |
| /services | 398 | Yes (2,799 chars) | Auto-static export |
| /asm | 2,631 | Yes (18,253 chars) | SSG (getStaticProps) |
| /asm/scoring-framework | 4,621 | Yes (33,987 chars) | SSG (getStaticProps) |
| /about | 423 | Yes (3,163 chars) | Auto-static export |
| /contact | 142 | Yes (966 chars) | Auto-static export |

### Navigation Without JavaScript

All primary navigation links use real `<a href>` tags with valid paths. Per-page link counts:

| Page | Total Links | Real hrefs | `href="#"` Placeholders |
|---|---|---|---|
| / | 23 | 20 | 3 |
| /services | 22 | 19 | 3 |
| /asm | 27 | 24 | 3 |
| /asm/scoring-framework | 23 | 20 | 3 |
| /about | 21 | 18 | 3 |
| /contact | 21 | 18 | 3 |

### Findings

| # | Finding | Severity |
|---|---|---|
| CS-1 | `<noscript>` tags exist but are empty on all pages — no fallback messaging | Low |
| CS-2 | Cloudflare obfuscates contact email via `data-cfemail` encoding — requires JS to decode | Medium |
| CS-3 | No phone number or physical address on any page | Info |

### Score Justification

≥95% of content present in initial HTML across all pages. Fully SSR/SSG. All navigation links are real `<a href>` tags. Docked for the Cloudflare email obfuscation (agents cannot read the contact email without JavaScript decoding) and empty `<noscript>` fallbacks.

---

## Dimension 2: Structural Legibility — 72/100 (Weight: 20%)

**Core question:** How easily can an agent identify the core layout — navigation, main content, sidebar, footer — directly from the DOM structure?

### Landmark Coverage

| Element | / | /services | /asm | /about | /contact |
|---|---|---|---|---|---|
| `<header>` | 1 | 1 | 1 | 1 | 1 |
| `<nav>` | 1 | 1 | 1 | 1 | 1 |
| `<main>` | 1 | 1 | 1 | 1 | 1 |
| `<footer>` | 1 | 1 | 1 | 1 | 1 |
| `<section>` | 2 | 3 | 5 | 5 | 1 |
| `<article>` | 0 | 0 | 0 | 0 | 0 |
| `<aside>` | 0 | 0 | 0 | 0 | 0 |

### Semantic-to-Generic Ratio

| Page | Semantic Elements | `<div>` + `<span>` | Ratio |
|---|---|---|---|
| / | 6 | 64 | 8.6% |
| /services | 7 | 56 | 11.1% |
| /asm | 9 | 71 | 11.3% |
| /about | 9 | 62 | 12.7% |
| /contact | 5 | 45 | 10.0% |

### Heading Hierarchy

Clean sequential hierarchy on all pages except /asm. Representative examples:

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

**/asm** — Dual h1 (issue)
```
h1: The Agentic Web Is Here. Is Your Site Ready?    ← page heading
  h2: Two Frameworks. One Problem.
  h2: Six Dimensions of Agent Readiness
    ...
h1: The Agentic Web: Why Your Website Is...          ← embedded markdown h1
  h2: A Business Case for Agent-Ready Infrastructure
    ...
  h2: Company [footer]
  h2: Developer Tools [footer]
```

### Findings

| # | Finding | Severity |
|---|---|---|
| SL-1 | Semantic-to-generic ratio 8.6–12.7% across all pages — heavily div-driven markup | Medium |
| SL-2 | Zero `aria-label` or `aria-labelledby` on any landmark element (multiple `<section>` tags are unlabeled and indistinguishable) | Medium |
| SL-3 | Two `<h1>` elements on /asm — embedded business case markdown introduces a second document heading | Medium |
| SL-4 | Footer `<h2>` headings ("Company", "Developer Tools") pollute the heading outline on every page | Low |
| SL-5 | No `<article>` or `<aside>` elements used anywhere | Low |

### Score Justification

All primary landmarks present. Heading hierarchy clean except /asm. `lang="en"` set on all pages. Semantic ratio falls in the 8–15% band (scoring tier 2). Deductions for missing landmark labels (important when a page has 5+ `<section>` elements with no way for an agent to distinguish them) and the dual h1 structural ambiguity.

---

## Dimension 3: Interactive Manifest Clarity — 78/100 (Weight: 20%)

**Core question:** Can an agent find and invoke the site's key actions — CTAs, forms, interactive controls — through the accessibility tree?

### Native Interactive Elements

| Element | / | /services | /asm | /about | /contact |
|---|---|---|---|---|---|
| `<button>` | 4 | 3 | 3 | 3 | 4 |
| `<a href>` | 23 | 22 | 27 | 21 | 21 |
| `<input>` | 2 | 0 | 0 | 0 | 2 |
| `<textarea>` | 1 | 0 | 0 | 0 | 1 |

### Accessible Name Coverage

- **All buttons have text content:** 0 empty buttons across all pages
- **All links have text content:** 0 empty links across all pages
- **Form labels properly associated:** 3 `<label>` elements with `for` attributes on / and /contact

### ARIA State Attributes

**Zero across all pages.** No `aria-expanded`, `aria-selected`, `aria-disabled`, `aria-pressed`, or `aria-checked` on any element.

### Findings

| # | Finding | Severity |
|---|---|---|
| IMC-1 | 3 `<a href="#">` placeholder links on every page (social media icons in footer — Twitter, LinkedIn, GitHub — all point nowhere) | Medium |
| IMC-2 | Zero ARIA state attributes site-wide — mobile menu toggle has no `aria-expanded`, dark mode button has no `aria-pressed` | Medium |
| IMC-3 | 2 `onclick` attributes on /asm page | Low |
| IMC-4 | Dark mode toggle SVGs missing `aria-hidden="true"` (2 per page) | Low |
| IMC-5 | Decorative arrow SVGs in links missing `aria-hidden="true"` on /services and /asm | Low |

### Score Justification

Strong native element usage — all CTAs use `<button>` or `<a>`, all forms use `<input>` and `<textarea>`. No "invisible CTA" pattern. All interactive elements have accessible names. Deductions for the 3 dead social links (an agent that clicks "Twitter" hits a dead end), the complete absence of ARIA state communication, and minor SVG accessibility gaps.

---

## Dimension 4: Data Extractability — 30/100 (Weight: 15%)

**Core question:** How well is business-critical data structured in semantic HTML that agents can accurately parse and extract?

### Structured Data

| Schema Type | Present |
|---|---|
| JSON-LD (any) | **No** — 0 blocks on any page |
| Organization | **No** |
| WebSite / WebPage | **No** |
| Service | **No** |
| BreadcrumbList | **No** |
| Microdata (itemscope/itemprop) | **No** — detected instances are inside code examples only |

### Semantic Data Elements

| Page | `<table>` | `<dl>` | `<ul>` | `<ol>` |
|---|---|---|---|---|
| / | 0 | 0 | 3 | 0 |
| /services | 0 | 0 | 2 | 0 |
| /asm | 0 | 2 | 6 | 2 |
| /asm/scoring-framework | 20 | 0 | 6 | 2 |
| /about | 0 | 0 | 2 | 0 |
| /contact | 0 | 0 | 2 | 0 |

### Table Accessibility (/asm/scoring-framework only)

- 20 tables with `<thead>` and `<th>` elements
- **0 `<caption>` elements** — no table has a programmatic title
- **0 `scope` attributes** — no `<th>` declares row/column scope

### Findings

| # | Finding | Severity |
|---|---|---|
| DE-1 | Zero JSON-LD structured data on any page — no Organization, WebSite, WebPage, Service, or BreadcrumbList schema | Critical |
| DE-2 | Zero microdata markup on any page — agents cannot programmatically extract typed business data | Critical |
| DE-3 | Zero `<time datetime>` elements — dates are not machine-readable | Medium |
| DE-4 | Tables on /asm/scoring-framework lack `<caption>` and `scope` attributes | Medium |
| DE-5 | `<dl>`/`<dt>`/`<dd>` used only on /asm (2 definition lists) — unused elsewhere for attribute data | Low |

### Score Justification

No structured data markup of any kind. An agent can read the text content but cannot programmatically extract Organization identity, service offerings, contact information, or page hierarchy as typed data. The content is human-readable but not machine-parseable beyond raw text extraction. Tables exist on the framework page but lack full metadata. The `<dl>` usage on /asm is a positive signal but isolated.

---

## Dimension 5: Navigation Traversability — 68/100 (Weight: 10%)

**Core question:** Can an agent systematically explore the site via static links without requiring complex JavaScript interactions?

### Sitemap Analysis

**Location:** /sitemap.xml (referenced in robots.txt)
**Total URLs:** 43
**Last modified:** 2025-06-15 (all pages — 8 months stale)

**Missing from sitemap:**
- /asm
- /asm/scoring-framework
- /asm/build-spec
- /showcase
- /showcase/johnny
- /showcase/repram
- /showcase/termlife

**Ghost URLs in sitemap** (not reachable via any static navigation link):
- /services/blockchain-development, /services/smart-contracts, /services/software-development, /services/system-architecture, /services/token-launch, /services/web3-development
- /solutions, /solutions/quellion, /solutions/repram, /solutions/web3
- /case-studies, /blog, /resources, /resources/developer-resources, /docs
- /examples, /examples/tdemodapp
- 17 of 22 tool pages

28 of 43 sitemap URLs are unreachable via static links from the audited pages.

### Breadcrumbs

| Page | Breadcrumb Nav | BreadcrumbList JSON-LD |
|---|---|---|
| /asm/scoring-framework | Yes (`<nav aria-label="Breadcrumb">`) | No |
| /asm/build-spec | Yes (`<nav aria-label="Breadcrumb">`) | No |
| All other pages | No | No |

### Findings

| # | Finding | Severity |
|---|---|---|
| NT-1 | Sitemap is 8 months stale — all lastmod dates are 2025-06-15 | High |
| NT-2 | Sitemap contains 28 phantom/unreachable URLs from an old site structure | High |
| NT-3 | Sitemap missing 7 live pages (/asm, /asm/scoring-framework, /asm/build-spec, /showcase/*) | High |
| NT-4 | Breadcrumbs only on 2 spec pages; no BreadcrumbList JSON-LD anywhere | Medium |
| NT-5 | 3 placeholder `href="#"` social links on every page | Medium |
| NT-6 | 1 "Learn More" generic link text on homepage | Low |
| NT-7 | robots.txt allows all crawlers, including AI agents | Pass |
| NT-8 | Maximum click depth from homepage: 2 | Pass |

### Score Justification

The site's link structure is good — shallow depth, all navigation uses static `<a href>` tags, no JavaScript-gated navigation. However, the sitemap is significantly stale and actively misleading: it lists pages that don't exist and omits pages that do. An agent relying on the sitemap for discovery would have an inaccurate picture of the site. No breadcrumb schema for the pages that have breadcrumb UI.

---

## Dimension 6: Agent Response Fitness — 76/100 (Weight: 10%)

**Core question:** When the site's content is read as plain text — stripped of all visual formatting — does it tell a coherent, useful story that an agent can summarize accurately?

### Plain Text Stream (Homepage, First 300 Characters)

```
Clocktower and Associates Clocktower Home Services ASM Tools
About Contact Us Open main menu Building the Future Through Innovation
Clocktower and Associates delivers enterprise-grade technology solutions,
custom software development, and strategic consulting to help your
business thrive in the digital age.
```

Company name at character position 0. Value proposition within the first 200 characters. Navigation chrome (15 words) sits between the brand and the heading but does not break coherence.

### Boilerplate Analysis

Header (16 words) + Footer (49 words) = **65 words of boilerplate** repeated on every page.

| Page | Total Words | Unique Content | Boilerplate Ratio |
|---|---|---|---|
| / | 241 | 176 | 27.0% |
| /services | 398 | 333 | 16.3% |
| /asm | 2,631 | 2,566 | 2.5% |
| /about | 423 | 358 | 15.4% |
| /contact | 142 | 77 | **45.8%** |

### Image Alt Text

All images have alt text. No empty or missing `alt` attributes.

| Image | Alt Text | Pages |
|---|---|---|
| Header logo | "Clocktower and Associates Logo" | All |
| Footer logo | "Clocktower and Associates company logo - clocktower icon representing technology innovation and consulting services" | All |
| About page logo | "Clocktower and Associates full company logo featuring clocktower icon and professional technology consulting branding" | /about |

### Findings

| # | Finding | Severity |
|---|---|---|
| ARF-1 | /contact boilerplate ratio 45.8% — nearly half the page is navigation and footer chrome | Medium |
| ARF-2 | Two `<h1>` elements on /asm create ambiguity in page topic identification | Medium |
| ARF-3 | Homepage boilerplate ratio 27% — thin content page with proportionally heavy chrome | Low |
| ARF-4 | "Open main menu" hamburger label leaks into the text stream on every page | Low |
| ARF-5 | Footer logo alt text is keyword-stuffed and repeated on every page | Low |
| ARF-6 | Company name and value proposition appear in first 200 characters | Pass |
| ARF-7 | No duplicate heading texts on any page | Pass |
| ARF-8 | All images have descriptive alt text | Pass |
| ARF-9 | Content reads as coherent narrative in DOM order on all pages | Pass |

### Score Justification

Text streams are mostly coherent and read well in linear order. Headings are descriptive and accurately summarize their sections. Content ordering is good — business-critical information appears early. Main deductions for the /contact page being nearly half boilerplate (an agent reading that page gets more chrome than content) and the dual-h1 ambiguity on /asm.

---

## Summary: Before and After

This audit was conducted during the same session that discovered and fixed two critical access barriers. The score difference tells the story:

### Before Fixes

| Barrier | Impact |
|---|---|
| Cloudflare managed robots.txt blocking all AI crawlers (ClaudeBot, GPTBot, etc.) | Agents never reach the site |
| `DarkModeProvider` returning `null` during SSR | Agents that reach the site get an empty HTML shell |

```
Pre-fix ASM = (0 × 0.25) + (72 × 0.20) + (78 × 0.20) + (30 × 0.15) + (68 × 0.10) + (76 × 0.10)
            = 48.9 → 49

Grade: D — Agent-Hostile
```

Content Survivability: **0/100.** Every page returned `<div id="__next"></div>` and nothing else. The site was functionally invisible to the agentic web.

### After Fixes

```
Post-fix ASM = (92 × 0.25) + (72 × 0.20) + (78 × 0.20) + (30 × 0.15) + (68 × 0.10) + (76 × 0.10)
             = 71.9 → 72

Grade: C — Agent-Impaired
```

Content Survivability: **92/100.** All pages serve full pre-rendered HTML. The site went from invisible to functional in a one-line code change.

### What Remains

The gap between C and B (75+) is primarily Data Extractability. Adding JSON-LD structured data (Organization, WebPage, Service, BreadcrumbList schemas) and updating the sitemap would push the score into the Agent-Functional range without any architectural changes.

---

## Top 5 Priority Actions

| Priority | Action | Dimensions | Expected Impact |
|---|---|---|---|
| 1 | **Add JSON-LD structured data** — Organization schema site-wide, WebPage per page, Service on /services, BreadcrumbList where breadcrumb UI exists | Data Extractability | +25–35 points on DE; ~+4–5 composite |
| 2 | **Rebuild sitemap.xml** — Add missing pages, remove phantom URLs, update lastmod dates, automate generation | Navigation Traversability | +15–20 points on NT; ~+2 composite |
| 3 | **Add ARIA landmark labels and state attributes** — `aria-label` on `<section>` elements, `aria-expanded` on mobile menu, `aria-pressed` on dark mode toggle | Structural Legibility, Interactive Manifest Clarity | +5–8 points across SL/IMC; ~+2 composite |
| 4 | **Fix the /asm dual h1** — Downgrade embedded business case h1 to h2 or suppress heading level in the rendering wrapper | Structural Legibility, Agent Response Fitness | +3–5 points across SL/ARF; ~+1 composite |
| 5 | **Replace or remove placeholder social links** — Either connect to real profiles or remove the dead `href="#"` links from the footer | Interactive Manifest Clarity, Navigation Traversability | +2–3 points across IMC/NT; ~+1 composite |

**Projected score after top 2 actions: ~79 — Grade B (Agent-Functional)**

---

*This report was generated using the ASM Scoring Framework v1.0 scoring methodology. The site audited is our own — because if you're going to sell agent-readiness audits, you should be able to pass one.*
