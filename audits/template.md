# AXIOM Audit Report: [SITE URL]

**Framework:** AXIOM Scoring Framework v3.1
**Date:** [YYYY-MM-DD]
**Auditor:** [Name / Organization]
**Pages Audited:** [List of pages, e.g. /, /products, /about, /contact]

---

## Composite Score

| | |
|---|---|
| **AXIOM Score** | **[XX] / 100** |
| **Grade** | **[A/B/C/D/F] — [Agent-Ready / Agent-Functional / Agent-Impaired / Agent-Hostile / Agent-Invisible]** |
| **Findings** | [X] Critical, [X] High, [X] Medium, [X] Low, [X] Info |

```
AXIOM = (CS × 0.25) + (SL × 0.20) + (IMC × 0.20) + (DE × 0.15) + (NT × 0.10) + (ARF × 0.10)
      = ([CS] × 0.25) + ([SL] × 0.20) + ([IMC] × 0.20) + ([DE] × 0.15) + ([NT] × 0.10) + ([ARF] × 0.10)
      = [X] + [X] + [X] + [X] + [X] + [X]
      = [TOTAL]
```

### Scorecard

| Dimension | Score | Weight | Weighted | Status |
|---|---|---|---|---|
| Content Survivability | [XX] | 25% | [XX.X] | [Pass/Warn/Fail] |
| Structural Legibility | [XX] | 20% | [XX.X] | [Pass/Warn/Fail] |
| Interactive Manifest Clarity | [XX] | 20% | [XX.X] | [Pass/Warn/Fail] |
| Data Extractability | [XX] | 15% | [XX.X] | [Pass/Warn/Fail] |
| Navigation Traversability | [XX] | 10% | [XX.X] | [Pass/Warn/Fail] |
| Agent Response Fitness | [XX] | 10% | [XX.X] | [Pass/Warn/Fail] |

Pass: >=70 | Warn: 40-69 | Fail: <40

---

## Dimension 1: Content Survivability — [XX]/100 (Weight: 25%)

**Core question:** What percentage of the site's meaningful content remains visible and functional without JavaScript execution?

### Test Results

<!-- Compare raw HTML (curl) response against fully rendered page for each audited page -->

| Page | Body Word Count | Content in Raw HTML | Rendering Method |
|---|---|---|---|
| / | [XX] | [Yes/No] ([X] chars) | [SSR/SSG/CSR/Hybrid] |
| [page] | [XX] | [Yes/No] ([X] chars) | [SSR/SSG/CSR/Hybrid] |

### Navigation Without JavaScript

| Page | Total Links | Real hrefs | `href="#"` Placeholders |
|---|---|---|---|
| / | [XX] | [XX] | [XX] |
| [page] | [XX] | [XX] | [XX] |

### Findings

| # | Finding | Severity |
|---|---|---|
| CS-1 | [Description] | [Critical/High/Medium/Low/Info] |

### Score Justification

[Explain how the score was determined based on the scoring bands in AXIOM Scoring Framework Section 3.1]

---

## Dimension 2: Structural Legibility — [XX]/100 (Weight: 20%)

**Core question:** How easily can an agent identify the core layout — navigation, main content, sidebar, footer — directly from the DOM structure?

### Landmark Coverage

| Element | / | [page] | [page] |
|---|---|---|---|
| `<header>` | [X] | [X] | [X] |
| `<nav>` | [X] | [X] | [X] |
| `<main>` | [X] | [X] | [X] |
| `<footer>` | [X] | [X] | [X] |
| `<section>` | [X] | [X] | [X] |
| `<article>` | [X] | [X] | [X] |
| `<aside>` | [X] | [X] | [X] |

### Semantic-to-Generic Ratio

| Page | Semantic Elements | `<div>` + `<span>` | Ratio |
|---|---|---|---|
| / | [XX] | [XX] | [XX]% |
| [page] | [XX] | [XX] | [XX]% |

### Heading Hierarchy

<!-- Document the heading structure for each page. Flag skipped levels, duplicate h1s, or missing h1. -->

```
[Page path]
h1: [heading text]
  h2: [heading text]
    h3: [heading text]
```

### Findings

| # | Finding | Severity |
|---|---|---|
| SL-1 | [Description] | [Critical/High/Medium/Low/Info] |

### Score Justification

[Explain how the score was determined based on the scoring bands in AXIOM Scoring Framework Section 3.2]

---

## Dimension 3: Interactive Manifest Clarity — [XX]/100 (Weight: 20%)

**Core question:** Can an agent find and invoke the site's key actions — CTAs, forms, interactive controls — through the accessibility tree?

### Native Interactive Elements

| Element | / | [page] | [page] |
|---|---|---|---|
| `<button>` | [X] | [X] | [X] |
| `<a href>` | [X] | [X] | [X] |
| `<input>` | [X] | [X] | [X] |
| `<select>` | [X] | [X] | [X] |
| `<textarea>` | [X] | [X] | [X] |

### Accessible Name Coverage

<!-- Report on percentage of interactive elements with accessible names (visible text, aria-label, aria-labelledby) -->

### ARIA State Attributes

<!-- Document presence of aria-expanded, aria-selected, aria-disabled, aria-pressed, aria-checked on stateful elements -->

### Findings

| # | Finding | Severity |
|---|---|---|
| IMC-1 | [Description] | [Critical/High/Medium/Low/Info] |

### Score Justification

[Explain how the score was determined based on the scoring bands in AXIOM Scoring Framework Section 3.3]

---

## Dimension 4: Data Extractability — [XX]/100 (Weight: 15%)

**Core question:** How well is business-critical data structured in semantic HTML that agents can accurately parse and extract?

### Structured Data

| Schema Type | Present | Pages |
|---|---|---|
| JSON-LD (any) | [Yes/No] | [pages] |
| Organization | [Yes/No] | [pages] |
| WebSite / WebPage | [Yes/No] | [pages] |
| Product / Service | [Yes/No] | [pages] |
| BreadcrumbList | [Yes/No] | [pages] |
| [Other relevant] | [Yes/No] | [pages] |

### Semantic Data Elements

| Page | `<table>` | `<dl>` | `<ul>` | `<ol>` |
|---|---|---|---|---|
| / | [X] | [X] | [X] | [X] |
| [page] | [X] | [X] | [X] | [X] |

### Findings

| # | Finding | Severity |
|---|---|---|
| DE-1 | [Description] | [Critical/High/Medium/Low/Info] |

### Score Justification

[Explain how the score was determined based on the scoring bands in AXIOM Scoring Framework Section 3.4]

---

## Dimension 5: Navigation Traversability — [XX]/100 (Weight: 10%)

**Core question:** Can an agent systematically explore the site via static links without requiring complex JavaScript interactions?

### Sitemap Analysis

| Property | Value |
|---|---|
| Location | [path or "Not found"] |
| Total URLs | [XX] |
| Last modified | [date or "Not set"] |
| Referenced in robots.txt | [Yes/No] |

<!-- Note any missing pages or phantom URLs -->

### Breadcrumbs

| Page | Breadcrumb Nav | BreadcrumbList JSON-LD |
|---|---|---|
| [page] | [Yes/No] | [Yes/No] |

### AI Crawler Access (robots.txt)

| User-Agent | Status |
|---|---|
| GPTBot | [Allowed/Blocked/Not mentioned] |
| ClaudeBot | [Allowed/Blocked/Not mentioned] |
| PerplexityBot | [Allowed/Blocked/Not mentioned] |
| Google-Extended | [Allowed/Blocked/Not mentioned] |

### AXIOM Manifest

| Property | Value |
|---|---|
| `axiom.json` present | [Yes/No] |
| `<link rel="axiom-manifest">` | [Yes/No] |
| `tier2_allowed` | [true/false/not set] |
| `tier3_allowed` | [true/false/not set] |

### Findings

| # | Finding | Severity |
|---|---|---|
| NT-1 | [Description] | [Critical/High/Medium/Low/Info] |

### Score Justification

[Explain how the score was determined based on the scoring bands in AXIOM Scoring Framework Section 3.5]

---

## Dimension 6: Agent Response Fitness — [XX]/100 (Weight: 10%)

**Core question:** When the site's content is read as plain text — stripped of all visual formatting — does it tell a coherent, useful story that an agent can summarize accurately?

### Plain Text Stream (Homepage, First 300 Characters)

```
[Paste the first ~300 characters of the homepage text content as read in DOM order]
```

### Boilerplate Analysis

| Page | Total Words | Unique Content | Boilerplate Ratio |
|---|---|---|---|
| / | [XX] | [XX] | [XX]% |
| [page] | [XX] | [XX] | [XX]% |

### Image Alt Text

<!-- Document alt text coverage: missing, empty, descriptive, keyword-stuffed -->

### Findings

| # | Finding | Severity |
|---|---|---|
| ARF-1 | [Description] | [Critical/High/Medium/Low/Info] |

### Score Justification

[Explain how the score was determined based on the scoring bands in AXIOM Scoring Framework Section 3.6]

---

## Top 5 Priority Actions

| Priority | Action | Dimensions | Expected Impact |
|---|---|---|---|
| 1 | [Description] | [Dimensions affected] | [Expected score impact] |
| 2 | [Description] | [Dimensions affected] | [Expected score impact] |
| 3 | [Description] | [Dimensions affected] | [Expected score impact] |
| 4 | [Description] | [Dimensions affected] | [Expected score impact] |
| 5 | [Description] | [Dimensions affected] | [Expected score impact] |

---

## Methodology

| | |
|---|---|
| **Framework** | AXIOM Scoring Framework v3.1 |
| **Tools** | [List tools used, e.g. Charlotte MCP, curl, browser DevTools] |
| **Pages** | [List of pages audited] |
| **Environment** | [Production / Staging / Local] |

---

*This report was generated using the AXIOM Scoring Framework v3.1 methodology.*
