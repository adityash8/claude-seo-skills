# SEO Audit: example.com
Date: 2025-03-01
Tool: Ahrefs

## Executive Summary
Health Score: 74/100
Critical Issues: 3
Revenue Opportunity: $8,000-$24,000/mo (based on $120 ARPU, 2% CVR, traffic recovery)

### Quick Wins (this week)
1. Fix 14 noindex pages that are receiving organic traffic (§6)
2. Update robots.txt to allow AI crawlers + block UTM params (§7)
3. Add Organization + BreadcrumbList schema to homepage (§13)

### High Impact (this month)
1. Refresh 6 content-decayed pages that lost >40% traffic YoY (§10)
2. Fix 23 under-linked pages in P4-20 with <3 internal links (§14)
3. Resolve 847 broken internal links across 12 high-traffic pages (§11)

### Ongoing
1. Monthly CWV monitoring — 8 pages currently failing LCP (§5)
2. CTR optimization — 12 keywords underperforming by >30% vs position benchmark (§17)
3. Backlink acquisition — DR flat for 6 months, refdomains declining (§15)

---

## Section Results

### §1 Manual Actions — PASS
No manual penalties detected.
-> https://search.google.com/search-console/manual-actions?resource_id=sc-domain%3Aexample.com

### §2 Global Redirect — PASS
http://example.com -> 301 -> https://example.com
https://www.example.com -> 301 -> https://example.com

### §3 Organic Traffic — WARN
Monthly organic clicks: 42,300 (down 8% MoM)
Branded share: 61% (approaching critical threshold)
Mobile: 58% | Desktop: 42%

### §4 MoM Growth — WARN
Average MoM growth: -2.1% (declining)
At current trajectory: 0.78x traffic in 12 months

### §5 Core Web Vitals — CRITICAL
8 pages failing LCP (>4s on mobile). Top offenders by traffic:
- /blog/best-practices — LCP 6.2s (hero image uncompressed, 2.4MB)
- /pricing — LCP 4.8s (render-blocking font load)

**Fix for /blog/best-practices:**
```html
<!-- Before -->
<img src="/images/hero.png">

<!-- After -->
<img src="/images/hero.webp" width="1200" height="630" loading="eager"
     srcset="/images/hero-400.webp 400w, /images/hero-800.webp 800w, /images/hero-1200.webp 1200w"
     sizes="(max-width: 768px) 100vw, 1200px">
```

### §6 Indexability — CRITICAL
14 noindex pages receiving organic traffic (combined: 3,200 clicks/mo)
Top offender: /blog/category/news — 890 clicks, noindex tag present

### §7 Robots.txt — WARN
Issues found:
- AI crawlers (GPTBot, ClaudeBot) not explicitly allowed
- UTM parameters not blocked (duplicate content risk)
- Sitemap not referenced

**Corrected robots.txt:**
```
User-agent: GPTBot
User-agent: ChatGPT-User
User-agent: ClaudeBot
User-agent: Claude-Web
User-agent: PerplexityBot
Allow: /

User-agent: *
Allow: /

Disallow: /*?*utm_*
Disallow: /*&utm_*
Disallow: /*?*fbclid=*
Disallow: /*&fbclid=*

Sitemap: https://example.com/sitemap_index.xml
```

### §8 XML Sitemaps — PASS
sitemap_index.xml present, references 3 child sitemaps.
12 high-traffic pages missing from sitemap — added to backlog.

### §9 Canonicalization — PASS
All indexed pages have self-canonicals. No cross-domain canonical issues.

### §10 Content Decay — CRITICAL
6 pages lost >40% traffic YoY. Top decayed pages:
| URL | Traffic Lost | Top Keyword | Action |
|-----|-------------|------------|--------|
| /blog/saas-metrics | -67% | "saas metrics" | Full refresh |
| /features/reporting | -52% | "reporting dashboard" | Update + FAQ |
| /blog/customer-success | -44% | "customer success tips" | Update stats |

### §11 Broken Links — CRITICAL
847 broken internal links across 12 pages.
Top pages with broken links: /resources (234), /blog (189), /integrations (143)
Top lost backlinks: 3 referring domains linking to /old-pricing (DR 45, 38, 31) — 301 to /pricing

### §12 HTTP Issues — PASS
No internal HTTP links found. HSTS header present.

### §13 Schema Markup — WARN
Homepage missing: Organization, SoftwareApplication, BreadcrumbList

**Generated JSON-LD for homepage:**
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Example",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png",
  "sameAs": [
    "https://twitter.com/example",
    "https://linkedin.com/company/example"
  ]
}
```

### §14 Internal Linking — WARN
23 pages in P4-20 with <3 internal links.
Top opportunity: /blog/saas-churn-reduction (P7, 2 internal links) — add links from /blog hub

### §15 Backlink Profile — WARN
DR: 52 (flat for 6 months)
Referring domains: 341 (down from 389 twelve months ago, -12%)
Anchor distribution: branded 64%, partial match 22%, naked URL 9%, generic 5%

### §16 Content Gap — WARN
Top competitor: competitor.com (DR 61, 2.1x our traffic)
Top gap keywords (they rank, we don't):
| Keyword | Volume | KD | Competitor Position |
|---------|--------|-----|---------------------|
| customer success platform | 2,900 | 52 | P3 |
| saas retention software | 1,600 | 44 | P5 |
| churn analysis tool | 880 | 38 | P2 |

### §17 Organic CTR — WARN
12 keywords underperforming vs position benchmarks (>30% below expected CTR):

| Keyword | Position | Actual CTR | Expected CTR | Gap |
|---------|----------|-----------|-------------|-----|
| customer success software | 4 | 4.2% | 8.5% | -51% |
| saas churn rate | 6 | 1.8% | 3.5% | -49% |
| customer health score | 3 | 8.1% | 13% | -38% |

**Title tag variants for "customer success software" (P4, 4.2% CTR -> target 8.5%):**
1. `Customer Success Software That Reduces Churn [2025]`
2. `#1 Customer Success Platform — Cut Churn by 34%`
3. `Customer Success Software: Why 500+ Teams Switched`

**Meta description:**
`Stop losing accounts to churn. Example's AI health scoring flags at-risk customers 30 days before renewal. Free trial, no credit card.`

---

## Priority Action Matrix

| Priority | Action | Effort | Impact | Owner |
|----------|--------|--------|--------|-------|
| P1 | Remove noindex from 14 traffic-bearing pages | Low | High | Dev |
| P1 | Fix 847 broken internal links | Medium | High | Dev |
| P1 | Update robots.txt | Low | Medium | Dev |
| P2 | Refresh 6 decayed blog posts | High | High | Content |
| P2 | Add schema to homepage + pricing | Low | Medium | Dev |
| P2 | Update title tags for 12 low-CTR keywords | Medium | Medium | Content |
| P3 | Add internal links to 23 under-linked pages | Medium | Medium | Content |
| P3 | Fix LCP on 8 failing pages | High | Medium | Dev |
| P4 | Recover 3 broken backlinks (301 redirects) | Low | Medium | Dev |
| P4 | Build content for 3 top content gap keywords | High | High | Content |

---

## GEO Deliverables

### Homepage TL;DR Block
```
TL;DR: Example helps B2B teams reduce churn by 34% using AI-powered health scoring (2024 customer data).
Unlike manual spreadsheet tracking, it flags at-risk accounts 30 days before renewal.
Best for: SaaS companies with 50-500 accounts managed by CS teams under 10.
Last verified: March 2025 · Source: Example
```

### AI Surface Priority (Top 5 Keywords)

| Keyword | Volume | Position | Query Type | AIO? | AI Score | Recommendation |
|---------|--------|----------|-----------|------|----------|----------------|
| customer success software | 4,400 | 8 | Commercial | Yes | 7/10 | TL;DR block + FAQ schema |
| churn prediction tool | 1,600 | 14 | Informational | Yes | 9/10 | GEO content refresh |
| customer health score | 2,900 | 5 | Informational | Yes | 8/10 | Featured snippet optimization |
| saas retention rate | 3,200 | 12 | Informational | No | 6/10 | Traditional SEO + stats |
| customer success platform | 2,900 | 22 | Commercial | Yes | 5/10 | Comparison page + schema |

---

*This is a sample report structure. Actual reports will contain real data from your Ahrefs/Semrush project.*
